---
layout: post
title: Create an ASP.NET MVC site on OSX (Detailed explanation)
---

![ASP NET](http://www-asp.azureedge.net/v-2015-12-16-016/images/ui/asplogo-square.png)

## Introduction
Microsoft has always been known for their closed ecosystem. From their frameworks (WebForms, MVC, Entity Framework) to their development tools (Visual Studio, Blend), the only platform you could use was always Windows. This made sense in the past when desktop computing was the way to go and Windows held a huge chunk of both mobile and desktop markets. But now, with more and more people using mobile devices over classical desktops (iPhones, iPads, Android phones & tablets, Chromebooks etc...) and with the emergence of high quality multi-platform development environments (Ruby on Rails, Node...), Windows development stack has been steadily losing their developers to more open platforms. 

So now, at the time of great changes in Microsoft (influenced by the new CEO Satya Nadella), the folks at Redmond are turning their heads towards open source development. The past year has seen quite a lot of interesting development in the open source area. Microsoft unveiled their latest multi-platform IDE - [Visual Studio Code](https://code.visualstudio.com/) which already has a vibrant community of extenders and collaborators at [OmniSharp](http://http://www.omnisharp.net/) . Microsoft has also developed a completely new development stack that is open source and multi-platform. Developed and maintained under the reins of the [.NET Foundation](http://www.dotnetfoundation.org/) and published on [GitHub](https://github.com/dotnet/core), the new stack is called simply **.NET Core** . Currently it powers the new [ASP.NET Core 1.0 / ASP.NET 5](http://docs.asp.net/en/latest/conceptual-overview/aspnet.html) (I'll explain the naming ambiguity later on). It's still not a fully featured framework, but it's definitely a step in the right direction. And one Microsoft should have made long time ago. Shame on you Mr. Ballmer.

![Shame on you Steve Ballmer](http://www.targotennisberg.com/tarkvara/wp-content/uploads/2012/10/ballmer.jpeg)

Every tutorial currently available uses some form of code generators or templating, adding bloatware to your web application. I hold the opinion that, if you want bloatware on your application, you will put it there yourself. So this tutorial takes a different approach, giving you all of the basics you need to create a fully functional ASP.NET MVC app and guiding you through the process one step at a time.

## Naming conventions and ambiguity
Even though they've been taking steps in the right directions, Microsoft has made some mistakes, mainly in the way they branded the whole new concept. To their credit, they are fixing it, but as it is now - some things might cause headaches for newcommers to the platform.

A great introductory point to everyone should be [This article](http://www.hanselman.com/blog/ASPNET5IsDeadIntroducingASPNETCore10AndNETCore10.aspx) by Scott Hanselman.

I don't want to waste too much of your time on this so the major changes can be summed up as this:
* ASP.NET 5 will be called ASP.NET Core 1.0
* .NET Core 5 will be called .NET Core 1.0
* Entity Framework 7 will be called Entity Framework Core 1.0 or EF Core 1.0 colloquialy

## Development enviroment 
Obviously, you cannot execute Windows compilers and files on Unix based systems. So the folks at the .NET Foundation went and recreated the entire compiler and the execution environment for OSX and Linux. It's called the .NET Execution Environment or DNX. As all C# apps run on the Common Language Runtime (CLR), DNX runs the host process, CLR hosting logic and managed entry point discovery. It enables developers to run not only ASP.NET apps, but all apps built on the .NET Core runtime. DNX, and it's components are shipped entirely as NuGet packages (for newcommers to the Microsoft ecosystem - NuGet is a package managing system. Find out more about it at [nuget.org](https://www.nuget.org/) . DNX is open source friendly, modular and dramatically simplifies the way projects and solutions (C# solutions are bundles of multiple projects and all dependencies) are built and maintained. 

## Installing ASP.NET 5/Core 1.0 on OSX

### Installing the .NET Version Manager (DNVM)
To install the DNVM on OSX:

1. Run the following curl command
{% highlight bash %}
curl -sSL https://raw.githubusercontent.com/aspnet/Home/dev/dnvminstall.sh | DNX_BRANCH=dev sh && source ~/.dnx/dnvm/dnvm.sh
{% endhighlight %}
2. Run `dnvm`for DNVM help and just to check that the install process went OK

### Install the .NET Execution Environment (DNX)
You have two choices when it comes to installing the DNX. You can either install the **.NET Core 1.0 ** or install **Mono** . Both are great and it would be hard to recommend one over the other, so I tend to install both of them and switch when I need to. However, since this entire post is based around Microsofts efforts to drive their frameworks to new platforms - I will focus on **.NET Core 1.0 ** .

If your DNVM is properly installed, you should be able to install ** .NET Core 1.0 ** via the following command: 

{% highlight bash %}
dnvm upgrade -r coreclr
{% endhighlight %}

Install **Mono** via:

{% highlight bash %}
	dnvm upgrade -r mono
{% endhighlight %}

Now your .NET environment should be installed and you should be ready to go!

## Creating your first DNX project

### Project structure
This is the part where Microsoft gets a lot of kudos from me. They have reduced the complex project xml files with weird hierarchies and obscure file systems to a simple, transparent solution. A DNX project is every folder that contains the **project.json** file. As simple as that! The name of the project is the name of the folder and all the files are included by default. If you'd like to exclude some files, you just specify it in the **project.json** file and you're good to go. 

The main parts of the **project.json** file are

* Project version
* Frameworks 
* Dependencies


You can also define tons of other metadata elements such as

* Project version (e.g. 1.2.1-* )
* Project description
* Project authors ( as a JSON list, e.g. ["Mislav", "John", "Francis"])
* Project URL, license URL

These are just some of the basic features many projects have. They enable you to create your projects as NuGet packages and easily manage dependencies

### Creating a simple Console App

In order to create a DNX project, follow these steps:

1. Create a folder in your file system (the name of the folder will be the project name)
2. Inside that folder, create a **project.json** file.

And as simple as that, you have the basic layout for a DNX project. Very simple and elegant, no code generators needed.

#### 2.1 Populating the project.json file 

Copy this into your project.json file:

{% highlight json %}
{
    "version": "1.0.0-*",
    "description": "DemoProject Console App",
    "authors": [
        "name one",
        "name two"
    ],
    
    "frameworks": {
        "dnxcore50" : {
            "dependencies": {
                "Microsoft.CSharp": "4.0.1-beta-23516",
                "System.Console": "4.0.0-beta-23516"
            }
        }
    }
}
{% endhighlight %}
Here you can see all the things I mentioned in the previous section. 

I beleive that the first three fields are self-explanatory so I won't be focusing on them. 

The main part of the project resides in the "frameworks" sections of the **project.json** file. The frameworks file defines which CLR runtime the DNX will use. I mentioned two of them already -  .NET Core CLR and Mono. But they are not alone in any way. There is a whole array of products based on the .NET runtime that you can use. You list them all as children of the "dependencies" field in **project.json** . These include web frameworks, Windows Phone, iOS & Android frameworks (e.g. Xamarin) and various others. For a full list, check out the [NuGet page](https://github.com/dotnet/corefx/blob/master/Documentation/project-docs/standard-platform.md#nuget) . We will focus on the DNX core runtime.

Once you have added the framework name under the "frameworks", you can add specific dependencies under the framework. Since **.NET Core** is completely modular, you have to define it's dependencies for everything. 
The dependencies I have defined in the **project.json** are as follows:

1. **Microsoft.CSharp** : A basic depedency that includes the most used features of the C# language (such as : Collections, Linq, Threading etc...)
2. **System.Console** : A dependecy for writing to the output stream. I added it so we can create a "Hello DNX" program that writes to the console

Note that these dependencies are very,very basic. Your usual DNX project will have a lot more. 

Dependencies are written in the format:
````
    "package_name" : "package_version"
````

If you want the dependecies to be applied to all of the frameworks you use (very useful for multiplatform projects that use different frameworks depending on the OS), just add the dependencies under the root object in `project.json`

**You can determine the package version by going to http://www.nuget.org and searching. For example you would search System.Console and get the result**

#### Downloading & Installing the dependencies

In order to download & install the dependencies, we use the `DNVM` (.NET version manager) that we installed at the beginning of this tutorial. 

In order to check all the available frameworks you have installed, run `dnvm list` command. It lists all the frameworks you have and their respective versions. For me, the output looks like this:

````
Active Version              Runtime Architecture OperatingSystem Alias
------ -------              ------- ------------ --------------- -----
  *    1.0.0-rc1-update1    coreclr x64          darwin          
       1.0.0-rc1-update1    mono                 linux/osx       default
       1.0.0-rc1-update1    clr     x64          win             
       1.0.0-rc1-update1    clr     x86          win             
       1.0.0-rc1-update1    coreclr x64          linux           
       1.0.0-rc1-update1    coreclr x64          win             
       1.0.0-rc1-update1    coreclr x86          win       
````

There will be an asterisk before the version you are currently using. In order for the compilation to work as it should in this tutorial, you need to set the used runtime to `coreclr` . If there is already an asterisk in front of the `coreclr` framework, then you don't need to do anything. In case there is an asterisk in front of some other framework, you need to change your framework. You do this by running the `dnvm use [version_number] -r CoreCLR` .
So taking my output into consideration, I would run `dnvm use 1.0.0-rc1-update1 -r CoreCLR` .

Using the terminal, position yourself into your project folder.

After you are done setting the framework, you need to let the `DNVM` download the dependencies (you must have an active internet connection, and let the `DNVM` through your firewall). You do this by running the `dnvm restore` command from your terminal.

Now let the `DNVM` finish the download. You should see some output with GET requests and HTML responses. 

If everything went well, your dependencies should be installed and you should see a `project.lock.json` file appear in your project folder.

#### Creating the app entry point

Every program written in the C# language starts at an entry point of type: `ProjectName/Program.cs/Main()`. Where `ProjectName` is the C# namespace, `Program.cs` is the C# class name and `Main()` is the C# function name. This is not a very strict rule, more of a convention. Since C# only looks for a function with the signature `public static void Main()` .So now, in your project directory, create a file called Program.cs. Inside of that file, add the following C# code:

{% highlight csharp %}
using System;

namespace DemoProject
{
    public class Program
    {
        public static void Main(string[] args)
        {
            Console.WriteLine("Hello DNX Core!");
        }
    }
}
{% endhighlight %}

**NOTE - Replace the DemoProject with the name of your project**

Save the `Program.cs` file and run the `dnx run` command. This command will compile and run your code. You should see a Console output that says `"Hello DNX Core!` . Congradulations, you just created your very first Console App using the .NET Core!

### Creating a basic ASP.NET app

**DON'T JUST SKIP TO THIS PART, THIS ASSUMES YOU'VE ALREADY WENT THROUGH THE SIMPLE CONSOLE APP SECTION**

In this section, we will reuse a lot of the concepts that we established in the "Simple Console App" project.


#### 1. Creating the folder and adding dependencies

Create a new folder and add a `project.json` file. Inside the `project.json` file, add these dependencies (if things aren't working correctly, check the version numbers on http://nuget.org ) : 

{% highlight json %}
{
  "version": "1.0.0-*",
  "compilationOptions": {
    "emitEntryPoint": true
  },

  "dependencies": {
    "Microsoft.AspNet.Mvc": "6.0.0-rc1-final",
    "Microsoft.AspNet.Mvc.TagHelpers": "6.0.0-rc1-final",
    "Microsoft.AspNet.Server.Kestrel": "1.0.0-rc1-final",
    "Microsoft.AspNet.StaticFiles": "1.0.0-rc1-final",
    "Microsoft.AspNet.Tooling.Razor": "1.0.0-rc1-final",
    "Microsoft.Extensions.CodeGenerators.Mvc": "1.0.0-rc1-final",
    "Microsoft.Extensions.Configuration.FileProviderExtensions" : "1.0.0-rc1-final",
    "Microsoft.Extensions.Configuration.Json": "1.0.0-rc1-final",
    "Microsoft.Extensions.Configuration.UserSecrets": "1.0.0-rc1-final",
    "Microsoft.Extensions.Logging": "1.0.0-rc1-final",
    "Microsoft.Extensions.Logging.Console": "1.0.0-rc1-final",
    "Microsoft.Extensions.Logging.Debug": "1.0.0-rc1-final",
    "Microsoft.VisualStudio.Web.BrowserLink.Loader": "14.0.0-rc1-final"
  },

  "commands": {
    "web": "Microsoft.AspNet.Server.Kestrel",
  },

  "frameworks": {
    "dnx451": { },
    "dnxcore50": { }
  },

  "exclude": [
    "wwwroot",
  ]
}
{% endhighlight %}

As I mentioned before, the `project.json` file has a ton of different options and I couldn't possibly cover all of them in this article. For now I will mention the new ones I included in this `project.json`.

- "commands" - this is a very useful tool built in. It enables you to make common commands shorter to write. For example - to run a server, you would have to type `dnx Microsoft.AspNet.Server.Kestrel` into your terminal. But this way, you reduced it to running `dnx web` . Commands is a very powerful DNX tool and you should use it as much as you can. 
- "exclude" - this excludes the files or folders that are mentioned here. You will understand why we excluded `wwwroot` very soon

Save the `project.json` file and create a `wwwroot` file in your project folder. This is a file where all of your static files will live (.css, .js, images etc.). 

Run `dnu restore` to see if everything went well

#### Creating the Startup.cs file

The Startup.cs file is the entry point into your application. 

Inside the Startup.cs file you will have to implement four methods:

1. Constructor - `public Startup(IHostingEnvironment env)`
2. Services configuration `public void ConfigureServices(IServiceCollection services)``
3. ASP App configuration `public void Configure(IApplicationBuilder app, IHostingEnvironment env)`
4. Entry point to the app - `public static void Main(string[] args)``

The file will end up looking like this:

{% highlight csharp %}

using Microsoft.AspNet.Builder;
using Microsoft.AspNet.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;


namespace DemoProject
{
   public class Startup
   {
       public Startup(IHostingEnvironment env)
       {
           var builder = new ConfigurationBuilder();
           Configuration = builder.Build();
           
       }
       
       public IConfiguration Configuration {get; set;}
       
       public void ConfigureServices(IServiceCollection services)
       {
           services.AddMvc();
       }
       
       public void Configure(IApplicationBuilder app, IHostingEnvironment env)
       {
           app.UseStaticFiles();
           
           app.UseMvc(routes =>{
               routes.MapRoute(
                   name: "default",
                   template: "{controller}/{action}/{id}",
                   defaults: new {
                       controller = "Home",
                       action = "Index",
                       id = ""
                   }
               );
           });
       }
       
       public static void Main(string[] args) => Microsoft.AspNet.Hosting.WebApplication.Run<Startup>(args);
   }
}
{% endhighlight %} 

**NOTE Replace DemoProject with your project name**

There is a lot of reasoning as to why the Startup file must look like this, but this article won't be going into that. What I will cover, however, is the MVC routing part. This is covered in the `public void Configure(...)` method.

The first part `app.useStaticFiles()` enables the browser to access the files within the `wwwroot` part of your application.

The second part `app.useMvc(...)` configures how your MVC routing will be handled by the server. The template that I used (`{controller}/{action}/{id}`)is the most common one and it states how the request will be routed. For example - if I made a request to "siteurl.eg/home/index" the app would look for the controller called `HomeController` and find the function called `Index(...)` . Than that function would be responsible for returning HTML to the browser. 

The `defaults` part of the `routes.MapRoute(...)` function just specifies which controller and which action to use when nothing is specified in the url (e.g. "http://yourdomain.com" would be the same as "http://yourdomain.com/home/index")

#### Implementing controllers and views

So as I mentioned earlier, the routing needs a controller. In order to achieve this, create a folder called `Controllers` in your project folder. Inside of that folder, create a file called `HomeController.cs`. Open that file and add the following:

{% highlight csharp %}

using Microsoft.AspNet.Mvc;

namespace DemoProject.Controllers
{
    public class HomeController : Controller
    {
        public IActionResult Index()
        {
            return View();
        }
    }
}
{% endhighlight %}

 **NOTE Replace DemoProject with your project name**
 
 What this code does is it creates a class called `HomeController` that inherits from the `Controller` class. Then it implements an `Action` called `Index`. The `Index` action returns an `IActionResult` which can be HTML or a string or any number of other objects. It is a response to a HTML request. 
 
 Inside the `Index()` action, we have implemented the `return View()` function. The `View()` function returns HTML to the browser. If the `View` function has no parameters, then it returns the `.cshtml` (cshtml is ASP.NETs file type that can have CS code with HTML and is parsed by Microsofts Razor engine) that follows the same folder structure as the Controller, but only starts from the `Views` folder in the project folder instead of the `Controllers` folder. 
 
 So let's create a `Views` folder inside of our project folder. Inside the `Views` folder, create the `Home` folder. Inside the `Home` folder, create a file called `Index.cshtml`. This will be the file that our controller will server. Inside of it, you can write any Razor syntax you want (Razor syntax is HTML + C#, but nothing prohibits you from using only HTML)
 
 Inside of the `Index.cshtml` file, write any HTML you want. I put this simple HTML snippet:
 

>\<div style="font-family: sans-serif"\>
>   Hello \<span style="color: #800">MVC!\</span\>    
>\</div\>

After you completed your `Index.cshtml` file, save it and run `dnx web` from your terminal. If everything went well, you should get a message like this:

````
Hosting environment: Production
Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
````
Try running `localhost:5000` or whatever port the server was assigned to and you should have your website running.

This is the no-bullshit way of generating your ASP.NET website on OSX. No code generators, no help. Use can use whatever text editor you want, but for ASP development, I recommend Visual Studio Code.

Hope you've learned something from this tutorial

<div id="disqus_thread"></div>
<script>
    /**
     *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
     *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables
     */
    /*
    var disqus_config = function () {
        this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
        this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
    };
    */
    (function() {  // DON'T EDIT BELOW THIS LINE
        var d = document, s = d.createElement('script');
        
        s.src = '//mislavjavor.disqus.com/embed.js';
        
        s.setAttribute('data-timestamp', +new Date());
        (d.head || d.body).appendChild(s);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>

<!-- hitwebcounter Code START -->
<a href="http://www.hitwebcounter.com" target="_blank">
<img src="http://hitwebcounter.com/counter/counter.php?page=6301643&style=0024&nbdigits=5&type=ip&initCount=0" title="" Alt=""   border="0" >
</a>                                        <br/>
                                        <!-- hitwebcounter.com --><a href="http://www.hitwebcounter.com" title="Live Stats For Website" 
                                        target="_blank" style="font-family: Geneva, Arial, Helvetica, sans-serif; 
                                        font-size: 10px; color: #908C86; text-decoration: underline ;"><em>Live Stats For Website                                        </em>
                                        </a>   
                                                             
