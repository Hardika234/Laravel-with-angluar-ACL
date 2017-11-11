# Laravel-with-angluar-ACL


For the Laravel Route redirection with user's role. it allows you to protect your route based on user's role.Here user's role is maintaining in database. from the database you can fetch the user's roles/permissions.
Then as per those records we compare with the angular routes. if its authorized user for the particular route activity it will
show the content or allow access.

# Example 

The current user has a role of "Users". User Role has privileges to access "dashboard", But Suposse Another department is "staff". now logged in user is not assigned to access the staff module he can only access the dashboard from the admin panel.So that staff option from the menu will be hidden on that User's panel. 

# Installation:

Laravel 5.4 Setup

# Step: 1
Create UsersController.php and write a query to get user's role. Before that please refer the database tables "user_permissions.sql" for that example. 

# Step:2
Edit routes/web.php file 
Create a genric funtion in web.php file

    ## Global permission. 
    function Globalpermission()
    {
        $UserPermission = array();
        if (Auth::check()) {
            $DepartmentId = Auth::user()->department_id;
            $Permissions  = array();
            ## To get permissions to the module.
            $UserPermission = App\UserPermissions::select('permission_modules.department_id', 'modules.name', 'permission_modules.module_id')
                ->UsersData()
                ->where('permission_modules.department_id', $DepartmentId)
                ->get()->toArray();

            foreach ($UserPermission as $key => $value) {
                $Permissions[] = "'" . $value['name'] . "'";
            }
            $UserPermission['permissions'] = implode(",", $Permissions);
            $UserPermission['role']        = $DepartmentId;
        }
        return $UserPermission;
    }

Now, Returned value pass the route.

    Route::middleware('ManageAuth')->get('/user', function () {
        return view('layouts.managemaster')->with('UserPermission', Globalpermission());
    });
    
# Step:3 
Create a layout/master.blade.php file to config with angularjs.
Add angular-acl.js file to your directory.

    {{ HTML::script('resources/assets/js/angular-acl.js')}}

And While your are requesting Laravel Routes you will get user's role.you have to pass this role with angularJs configuration.
This code to layout/master.blade.php File.

    <script type="text/javascript">
            app.run(['AclService', function (AclService) {
                    var aclData = { {{ $UserPermission['role']}}:[<?=$UserPermission['permissions'];?>] }
                      AclService.setAbilities(aclData);
                      // Attach the mbember role to the current user
                     AclService.attachRole('{{ $UserPermission['role'] }}');
            }]);
        </script>

# Step: 4

Create angularjs file to customize application.
Add dependecy to acl angular routes.
    
    var app = angular.module('MyApp', ['ngRoute','mm.acl']);
    
Define angular routes.

    app.config(['$routeProvider', '$locationProvider',
        function($routeProvider, $locationProvider) {
            $routeProvider.
            when('/manage/dashboard', {
                templateUrl: 'resources/views/template path',
                controller: 'Controllername',
                resolve: {
                    'acl': function(AclService, $q) {
                        if (AclService.can('Role Name to assign route')) {
                            // Has proper permissions
                            return true;
                        } else {
                            // Does not have permission
                            return $q.reject('Unauthorized');
                        }
                    }
                }
            }).
            when('/manage/user', {
                templateUrl: 'resources/views/template path',
                controller: 'Controllername',
                resolve: {
                    'acl': function(AclService, $q) {
                        if (AclService.can('Role Name to assign route')) {
                            // Has proper permissions
                            return true;
                        } else {
                            // Does not have permission
                            return $q.reject('Unauthorized');
                        }
                    }
                }
            }).otherwise({
                redirectTo: '/manage/dashboard'
            });
            $locationProvider.html5Mode(true);
        }
    ]);
    
    app.run(['$rootScope', '$location', function ($rootScope, $location) {
      // If the route change failed due to our "Unauthorized" error, redirect them 
      $rootScope.$on('$routeChangeError', function(current, previous, rejection){
        if(rejection === 'Unauthorized'){
          $location.path('/');
        }
      })
    }]);
    
Edit your template file. Ex.

    <a ng-href="user/{{ id }}" ng-show="can('user')">Users</a>
    <a ng-href="staff/{{ id }}" ng-show="can('staff')">Staff</a>
       


