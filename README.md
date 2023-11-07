# Week-9 Lab-Manual (Authorization and External Sign-on)
This repository contains instructions that you may need to navigate creating Authorization for Web Applications

## Application Set up

1.	Create an MVC app. Choose Individual Accounts from the Authentication type dropdown before creating the application.
2.	Run Update-Database in pm console.
3.	Open SQL Server Object Explorer and verify your database is present (only for windows).
4.	Then Add > scaffold item > Identity > Choose all pages
5.	Register a few users and try logging in with them.


## Part 1 - Role Based Authentication

1. Update your *builder.Services.AddDefaultIdentity* line so that it looks as shown below. You will need to include *AddRoles*:

```
builder.Services.AddDefaultIdentity<IdentityUser>(options => options.SignIn.RequireConfirmedAccount = true)
    .AddRoles<IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>();

```

2. Let us add roles to our existing application by adding the below piece of code to your Program.cs. Modify the roles according to your requirements.

```
app.MapRazorPages();
//Code to copy begins here
using(var scope = app.Services.CreateScope())
{
    var roleManager = scope.ServiceProvider.GetRequiredService<RoleManager<IdentityRole>>();
    var roles = new[] { "Admin", "Manager", "Member" };
    foreach (var role in roles)
    {
        if(!await roleManager.RoleExistsAsync(role))
        {
            await roleManager.CreateAsync(new IdentityRole(role));
        }
    }
}
//Code to copy ends here
app.Run();

```

3. Run the application, then open the AspNetRoles table from SQL Server Object Explorer and validate that the roles have been added to the tables. 

4. Then, open the AspNetUserRoles table. This must be empty.You can add mapping between users and roles in this table by entering role_id and user_id. Each user can have multiple roles as well. 

5. Go to the HomeController. We would like the Privacy page to be accessed only by Admins. So above the Privacy method add the line:

```
        [Authorize(Roles ="Admin")]
```
This will restrict other users from viewing the Privacy page. 

Useful Links:
- [Microsoft Documentation](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/roles?view=aspnetcore-7.0)
- [Youtube Tutorial](https://www.youtube.com/watch?v=Y6DCP-yH-9Q)


## Part 2- Adding Google Signon

### Setting up Google API

1. Login to your google account and visit [Google API & Services](https://console.cloud.google.com/apis/)

2. Setup OAuth
    - In the Oauth consent screen of the Dashboard:

    - Select User Type - External and CREATE.
    - In the App information dialog, Provide an app name for the app, user support email, and developer contact information.
    - Step through the Scopes step.
    - Step through the Test users step.
    - Review the OAuth consent screen and go back to the app Dashboard. 
3. In the Credentials tab of the application Dashboard, select CREATE CREDENTIALS > OAuth client ID.

4. Select Application type > Web application, choose a name.

5. In the **Authorized redirect URIs** section, select **ADD URI** to set the redirect URI. Example redirect URI: https://localhost:{PORT}/signin-google, where the {PORT} placeholder is the app's port.

6. Select the CREATE button.

7. Save the Client ID and Client Secret for use in the app's configuration. You may be prompted to download a JSON file. If so, download and keep it in a safe location. 

### Setting Up the Location Web App
1. Add Microsoft.AspNetCore.Authentication.Google NuGet package. 

2. Open Terminal and type the command
```
dotnet user-secrets init
```

Then, click on your project name and verify that \<UserSecretsId> has been added to propertygroup.

3. In the terminal, type the following commands to register your google client ID with the application:

```
dotnet user-secrets set "Authentication:Google:ClientId" "<client-id>"

dotnet user-secrets set "Authentication:Google:ClientSecret" "<client-secret>"
```

You will be able to find the client-id and client-secret from the JSON you downloaded. Alternatively, you can go to your google OAuth dashboard and redownload the file in case you lost it. 

Run the commands one-by-one. 

4. Right-click on project, choose Manage User Secrets and verify that the details you entered are present. 

5. Add the below code to your Program.cs to add google authentication to your application. 
```
builder.Services.AddAuthentication().AddGoogle(googleOptions =>
{
    googleOptions.ClientId = builder.Configuration["Authentication:Google:ClientId"];
    googleOptions.ClientSecret = builder.Configuration["Authentication:Google:ClientSecret"];
});
```

6. Run the app and select Log in. An option to sign in with Google appears.

7. After logging in with google, you may have to go to your database and set EmailConfirmed to true if your email service is not configured yet. 

Sources:
- [Google external login](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/social/google-logins?view=aspnetcore-7.0)

- [Enable secret storage](https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets?view=aspnetcore-7.0&tabs=windows#enable-secret-storage)






