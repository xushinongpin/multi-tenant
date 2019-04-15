php artisan make:command DeleteTenant 【 生成所在地址 app/Console/Commands/DeleteTenant.php 】

```
<?php
namespace App\Console\Commands;
use Hyn\Tenancy\Contracts\Repositories\HostnameRepository;
use Hyn\Tenancy\Contracts\Repositories\WebsiteRepository;
use Hyn\Tenancy\Environment;
use Hyn\Tenancy\Models\Hostname;
use Hyn\Tenancy\Models\Website;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Config;
class DeleteTenant extends Command
{
    protected $signature = 'tenant:delete {tenantname}';
    protected $description = 'Deletes a tenant of the provided name. Only available on the local environment e.g. php artisan tenant:delete cafejohn';
    public function handle()
    {
        // because this is a destructive command, we'll only allow to run this command
        // if you are on the local environment
        if (!app()->isLocal()) {
            $this->error('This command is only avilable on the local environment.');
            return;
        }
        $tenantname = $this->argument('tenantname');
        $this->deleteTenant($tenantname);
    }
    private function deleteTenant($tenantname)
    {
        $baseUrl = config('app.url_base');
        $fqdn = "{$tenantname}.{$baseUrl}";

        if ($hostname = Hostname::where('fqdn', $fqdn)->firstOrFail()) {
            $website = Website::where('id', $hostname->website_id)->firstOrFail();
            app(HostnameRepository::class)->delete($hostname, true);
            app(WebsiteRepository::class)->delete($website, true);
            $this->info("Tenant {$tenantname} successfully deleted.");
        }
    }
}
```

删除前发布资产

```
php artisan vendor:publish
  选择： Provider: Hyn\Tenancy\Providers\Tenants\ConfigurationProvider 这一行的数字键
  Which provider or tag's files would you like to publish?:
   [0 ] Publish files from all providers and tags listed below
   [1 ] Provider: BeyondCode\DumpServer\DumpServerServiceProvider
   [2 ] Provider: Fideloper\Proxy\TrustedProxyServiceProvider
   [3 ] Provider: Hyn\Tenancy\Providers\TenancyProvider
   [4 ] Provider: Hyn\Tenancy\Providers\Tenants\ConfigurationProvider
   [5 ] Provider: Hyn\Tenancy\Providers\WebserverProvider
   [6 ] Provider: Illuminate\Foundation\Providers\FoundationServiceProvider
   [7 ] Provider: Illuminate\Mail\MailServiceProvider
   [8 ] Provider: Illuminate\Notifications\NotificationServiceProvider
   [9 ] Provider: Illuminate\Pagination\PaginationServiceProvider
   [10] Provider: Laravel\Tinker\TinkerServiceProvider
   [11] Tag: config
   [12] Tag: laravel-errors
   [13] Tag: laravel-mail
   [14] Tag: laravel-notifications
   [15] Tag: laravel-pagination
   [16] Tag: tenancy
```

config/tenancy.php 配置

```
...
        'auto-delete-tenant-directory' => env('TENANCY_DIRECTORY_AUTO_DELETE', false),
...
        'auto-delete-tenant-database' => env('TENANCY_DATABASE_AUTO_DELETE', false),
...
        'auto-delete-tenant-database-user' => env('TENANCY_DATABASE_AUTO_DELETE_USER', false),
```

.env 配置

```
TENANCY_DIRECTORY_AUTO_DELETE=true
TENANCY_DATABASE_AUTO_DELETE=true
TENANCY_DATABASE_AUTO_DELETE_USER=true
```

执行删除命令

```
php artisan tenant:delete 你要删的
```



