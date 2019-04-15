安装 laravel-permission

```
composer require spatie/laravel-permission
```

发布资产 【 选择 Provider: Spatie\Permission\PermissionServiceProvider 】

```
php artisan vendor:publish
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
      [11] Provider: Spatie\Permission\PermissionServiceProvider
      [12] Tag: config
      [13] Tag: laravel-errors
      [14] Tag: laravel-mail
      [15] Tag: laravel-notifications
      [16] Tag: laravel-pagination
      [17] Tag: migrations
      [18] Tag: tenancy
```

修改 config/permission.php

```
    'permission' => Spatie\Permission\Models\Permission::class, 改为  'permission' => App\Permission::class,
    'role' => Spatie\Permission\Models\Role::class, 改为 'role' => App\Role::class,
```

生产model

```
    php artisan make:model Permission
    php artisan make:model Role
```

修改 app/Permission.php 与 app/Role.php

```
<?php
    namespace App;
    use Hyn\Tenancy\Traits\UsesTenantConnection;
    use Spatie\Permission\Models\Permission as BasePermission;
    class Permission extends BasePermission
    {
        use UsesTenantConnection;
    }
```

```
<?php
    namespace App;
    use Hyn\Tenancy\Traits\UsesTenantConnection;
    use Spatie\Permission\Models\Role as BaseRole;
    class Role extends BaseRole
    {
        use UsesTenantConnection;
    }
```

设置默认角色和权限 【 config / tenancy.php 】

```
'tenant-seed-class' => false, 改为 'tenant-seed-class' => TenantDatabaseSeeder::class,
```

修改 TenantDatabaseSeeder 【 位置：database/seeds/TenantDatabaseSeeder.php 】

