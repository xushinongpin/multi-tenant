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



