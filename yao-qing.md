创建一个空的通知

```
php artisan make:notification TenantCreated

app/Notifications/TenantCreated
【 为了安全，此处强制使用https 】
<?php
namespace App\Notifications;
use Illuminate\Notifications\Messages\MailMessage;
use Illuminate\Notifications\Notification;
use Illuminate\Support\Facades\Password;
class TenantCreated extends Notification
{
    private $hostname;
    public function __construct($hostname)
    {
        $this->hostname = $hostname;
    }
    public function via()
    {
        return ['mail'];
    }
    public function toMail($notifiable)
    {
        $token = Password::broker()->createToken($notifiable);
        $resetUrl = "https://{$this->hostname->fqdn}/password/reset/{$token}";
        $app = config('app.name');
        return (new MailMessage())
            ->subject("{$app} Invitation")
            ->greeting("Hello {$notifiable->name},")
            ->line("You have been invited to use {$app}!")
            ->line('To get started you need to set a password.')
            ->action('Set password', $resetUrl);
    }
}
```

创建租户时通知管理员

```
app/Console/Commands/CreateTenant.php

app/Console/Commands/CreateTenant.php:20

...
use App\Notifications\TenantCreated;

public function handle(){
        ...
                // we'll create a random secure password for our to-be admin
                $password = str_random();
                $this->addAdmin($name, $email, $password)->notify(new TenantCreated($tenant["hostname"]));
                $this->info("Tenant '{$tenantname}' is created and is now accessible at {$tenant["hostname"]->fqdn}");
                // $this->info("Admin {$email} can log in using password {$password}");
                $this->info("Admin {$email} has been invited!");
        ...
}
```

使用Auth

```
php artisan make:auth
```

创建中间件

```
php artisan make:middleware EnforceTenancy

app/Http/Middlware/EnforceTenancy.php

<?php
namespace App\Http\Middleware;
use Closure;
use Illuminate\Support\Facades\Config;
class EnforceTenancy
{
    public function handle($request, Closure $next)
    {
        Config::set('database.default', 'tenant');
        return $next($request);
    }
}
```

分配中间件

app/Http/Kernel.php注册中间件

```
app / Http / Kernel.php

...
protected $routeMiddleware = [
    ...
    'tenancy.enforce' => \App\Http\Middleware\EnforceTenancy::class,
];
...
```

routes/web.php在Auth::routes\(\);该分配的中间件

```
routes/web.php

...
Route::group(['middleware' => 'tenancy.enforce'], function () {
    Auth::routes();
});
...
```

对租户进行分类

```
php artisan make:model Tenant

app/Tenant.php
<?php
namespace App;
use Hyn\Tenancy\Environment;
use Hyn\Tenancy\Models\Hostname;
use Hyn\Tenancy\Models\Website;
use Illuminate\Support\Facades\Hash;
use Hyn\Tenancy\Contracts\Repositories\HostnameRepository;
use Hyn\Tenancy\Contracts\Repositories\WebsiteRepository;
use Illuminate\Support\Facades\Config;

/**
 * @property Website website
 * @property Hostname hostname
 * @property User admin
 */
class Tenant
{
    public function __construct($tenantname, User $admin = null)
    {
        $baseUrl = config('app.url_base');
        $fqdn = "{$tenantname}.{$baseUrl}";

        if ($this->hostname = Hostname::where('fqdn', $fqdn)->firstOrFail()) {
            $this->website = Website::where('id', $this->hostname->website_id)->firstOrFail();
        }

        $this->admin = $admin;
    }
    public function delete()
    {
        app(HostnameRepository::class)->delete($this->hostname, true);
        app(WebsiteRepository::class)->delete($this->website, true);
    }
    public static function createFrom($name, $email, $tenantname): Tenant
    {
        // create a website
        $website = new Website;
        app(WebsiteRepository::class)->create($website);
        // associate the website with a hostname
        $hostname = new Hostname;
        $baseUrl = config('app.url_base');
        $hostname->fqdn = "{$tenantname}.{$baseUrl}";
        app(HostnameRepository::class)->attach($hostname, $website);
        // make hostname current
        app(Environment::class)->tenant($website);
        $admin = static::makeAdmin($name, $email, str_random());

        return new Tenant($tenantname, $admin);
    }
    private static function makeAdmin($name, $email, $password): User
    {
        $admin = User::create(['name' => $name, 'email' => $email, 'password' => Hash::make($password)]);
        $admin->guard_name = 'web';
        $admin->assignRole('admin');
        return $admin;
    }

    public static function retrieveBy($tenantname): ?Tenant
    {
        $baseUrl = config('app.url_base');
        $fqdn = "{$tenantname}.{$baseUrl}";
        if (Hostname::where('fqdn', $fqdn)->exists()) {
            return new Tenant($tenantname);
        }
        return null;
    }
}
```

```
app/Console/Commands/CreateTenant.php
<?php
namespace App\Console\Commands;
use App\User;
use Hyn\Tenancy\Contracts\Repositories\HostnameRepository;
use Hyn\Tenancy\Contracts\Repositories\WebsiteRepository;
use Hyn\Tenancy\Environment;
use Hyn\Tenancy\Models\Hostname;
use Hyn\Tenancy\Models\Website;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Hash;
use Spatie\Permission\Models\Role;
use App\Notifications\TenantCreated;
use App\Tenant;
class CreateTenant extends Command
{
    protected $signature = 'tenant:create {name} {email} {tenantname}';
    protected $description = 'Creates a tenant with the provided name and email address e.g. php artisan tenant:create boise boise@example.com';
    public function handle()
    {
        $name = $this->argument('name');
        $email = $this->argument('email');
        $tenantname = $this->argument('tenantname');
        if ($this->tenantExists($tenantname)) {
            $this->error("A tenant with name '{$tenantname}' already exists.");
            return;
        }
        $tenant = Tenant::createFrom($name, $email, $tenantname);
        $this->info("Tenant '{$tenantname}' is created and is now accessible at {$tenant->hostname->fqdn}");
        // invite admin
        $tenant->admin->notify(new TenantCreated($tenant->hostname));
        $this->info("Admin {$email} has been invited!");
    }
    private function tenantExists($tenantname)
    {
        $baseUrl = config('app.url_base');
        $fqdn = "{$tenantname}.{$baseUrl}";
        return Hostname::where('fqdn', $fqdn)->exists();
    }
}
```

```
app/Console/Commands/DeleteTenant.php
<?php
namespace App\Console\Commands;
use Hyn\Tenancy\Contracts\Repositories\HostnameRepository;
use Hyn\Tenancy\Contracts\Repositories\WebsiteRepository;
use Hyn\Tenancy\Environment;
use Hyn\Tenancy\Models\Hostname;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Config;
use App\Tenant;
class DeleteTenant extends Command
{
    protected $signature = 'tenant:delete {tenantname}';
    protected $description = 'Deletes a tenant of the provided name. Only available on the local environment e.g. php artisan tenant:delete boise';
    public function handle()
    {
        // because this is a destructive command, we'll only allow to run this command
        // if you are on the local or testing
        if (!(app()->isLocal() || app()->runningUnitTests())) {
            $this->error('This command is only avilable on the local environment.');
            return;
        }
        $tenantname = $this->argument('tenantname');
        if ($tenant = Tenant::retrieveBy($tenantname)) {
            $tenant->delete();
            $this->info("Tenant {$tenantname} successfully deleted.");
        } else {
            $this->error("Couldn't find tenant {$tenantname}");
        }
    }
}
```

.env 邮箱配置可参考 [.env配置](https://laravel10.wordpress.com/2015/02/22/%E3%83%A1%E3%83%BC%E3%83%AB%E3%81%AE%E7%92%B0%E5%A2%83%E8%A8%AD%E5%AE%9A/)  与 [测试邮箱](https://mailtrap.io)

```

MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=7f1587d6342ac9
MAIL_PASSWORD=0481a0141dac28
MAIL_ENCRYPTION=null
```



