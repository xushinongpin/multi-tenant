php artisan make:command CreateTenant 【生产位置 app/Console/Commands/CreateTenant.php】

```
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

class CreateTenant extends Command
{
    protected $signature = 'tenant:create {name} {email} {tenantname}';
    protected $description = 'Creates a tenant with the provided name and email address e.g. php artisan tenant:create john john@example.com cafejohn';
    public function handle()
    {
        $name = $this->argument('name');
        $email = $this->argument('email');
        $tenantname = $this->argument('tenantname');
        if ($this->tenantExists($tenantname)) {
            $this->error("A tenant with name '{$tenantname}' already exists.");
            return;
        }
        $tenant = $this->registerTenant($name, $email, $tenantname);
        app(Environment::class)->tenant($tenant["website"]);

        // we'll create a random secure password for our to-be admin
        $password = str_random();
        $this->addAdmin($name, $email, $password);
        $this->info("Tenant '{$tenantname}' is created and is now accessible at {$tenant["hostname"]->fqdn}");
        $this->info("Admin {$email} can log in using password {$password}");
    }
    private function tenantExists($tenantname)
    {
        $baseUrl = config('app.url_base');
        $fqdn = "{$tenantname}.{$baseUrl}";
        return Hostname::where('fqdn', $fqdn)->exists();
    }
    private function registerTenant($name, $email, $tenantname)
    {
        // create a website
        $website = new Website;
        app(WebsiteRepository::class)->create($website);
        // associate the website with a hostname
        $hostname = new Hostname;
        $baseUrl = config('app.url_base');
        $hostname->fqdn = "{$tenantname}.{$baseUrl}";
        app(HostnameRepository::class)->attach($hostname, $website);
        return ["hostname"=>$hostname, "website"=>$website];
    }
    private function addAdmin($name, $email, $password)
    {
        $admin = User::create(['name' => $name, 'email' => $email, 'password' => Hash::make($password)]);
        return $admin;
    }
}
```

为主机名的FQDN分配基本URL

```
// add a new new key in config/app.php:
    'url_base' => env('APP_URL_BASE', 'http://localhost'),

// modify your .env file:
    APP_URL_BASE=townhouse.dev
    APP_URL=http://${APP_URL_BASE}
```

使用提供的名称和电子邮件地址创建租户

```
e.g. php artisan tenant:create john john@example.com cafejohn
```



