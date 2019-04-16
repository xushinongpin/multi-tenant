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

```
<?php
    use App\Permission;
    use Illuminate\Database\Seeder;
    use Spatie\Permission\Models\Role;
    class TenantDatabaseSeeder extends Seeder
    {
        public function run()
        {
            $this->addRolesAndPermissions();
        }
        private function addRolesAndPermissions()
        {
            // create permissions for an admin
            $adminPermissions = collect(['create user', 'edit user', 'delete user'])->map(function ($name) {
                return Permission::create(['name' => $name]);
            });
            // add admin role
            $adminRole = Role::create(['name' => 'admin']);
            $adminRole->givePermissionTo($adminPermissions);
            // add a default user role
            Role::create(['name' => 'user']);
        }
    }
```

授予管理用户管理权限 【 app/User.php 】

```
use Hyn\Tenancy\Traits\UsesTenantConnection;
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use UsesTenantConnection;
    use HasRoles;

    // ...
}
```

修改权限设置代码 【 app/Console/Commands/CreateTenant.php:53 】

```
    private function addAdmin($name, $email, $password)
    {
        $admin = User::create(['name' => $name, 'email' => $email, 'password' => Hash::make($password)]);
        $admin->guard_name = 'web';
        $admin->assignRole('admin');
        return $admin;
    }
```

将 database\migrations\2019\_04\_15\_163057\_create\_permission\_tables.php 迁移到 database\migrations\tenant\2019\_04\_15\_163057\_create\_permission\_tables.php

然后执行生产命令

```
php artisan tenant:create zlc zlc@lvtian.vip zlc
//删除命令
php artisan tenant:delete zlc
```

执行报错提示  Spatie\Permission\Exceptions\RoleDoesNotExist  : There is no role named \`admin\`.

```
临时解决方案
app/Console/Commands/CreateTenant.php:56
    private function addAdmin($name, $email, $password)
    {
        $admin = User::create(['name' => $name, 'email' => $email, 'password' => Hash::make($password)]);
        $admin->guard_name = 'web';
        $admin->assignRole('admin'); 改为 $admin->hasRole('admin');
        return $admin;
    }
    
assignRole --- Assign the given role to the model. ---  将给定角色分配给模型。
hasRole    --- Determine if the model has (one of) the given role(s).  --- 确定模型是否具有（一个）给定角色。
```



