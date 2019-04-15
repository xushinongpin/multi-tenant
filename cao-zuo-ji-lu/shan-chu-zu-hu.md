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



