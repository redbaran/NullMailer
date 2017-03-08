# NullDesk Email Extensions

## Overview

Email extension service packages for quickly integrating common mail scenarios into any .Net Core or .Net project using a variety of message delivery frameworks and cloud service providers.

Easily configure your application for different email services at startup based on environment, deployment configuration, or runtime detection of available mail providers.

## Status

|                                   |   |   |
|-----------------------------------|:-:|:-:|
|Project  [Issue Board](https://github.com/NullDesk/NullMailer/issues#boards?repos=79507993)|[![Build status](https://ci.appveyor.com/api/projects/status/5uc95cb6xho4qtdh/branch/master?svg=true)](https://ci.appveyor.com/project/StephenRedd/nullmailer/branch/master)|[![ZenHub](https://img.shields.io/badge/Shipping_faster_with-ZenHub-5e60ba.svg?style=flat-square)](https://github.com/NullDesk/NullMailer/issues#boards?repos=79507993)|
|NullDesk.Extensions.Mailer                                                                 |[![MyGet](https://img.shields.io/myget/nulldesk-ci/vpre/NullDesk.Extensions.Mailer.Core.svg)](https://www.myget.org/feed/nulldesk-ci/package/nuget/NullDesk.Extensions.Mailer.Core)|[![NuGet](https://img.shields.io/nuget/v/NullDesk.Extensions.Mailer.Core.svg)](https://www.nuget.org/packages/NullDesk.Extensions.Mailer.Core/)|
|NullDesk.Extensions.Mailer.MailKit                                                         |[![MyGet](https://img.shields.io/myget/nulldesk-ci/vpre/NullDesk.Extensions.Mailer.MailKit.svg)](https://www.myget.org/feed/nulldesk-ci/package/nuget/NullDesk.Extensions.Mailer.MailKit)|[![NuGet](https://img.shields.io/nuget/v/NullDesk.Extensions.Mailer.MailKit.svg)](https://www.nuget.org/packages/NullDesk.Extensions.Mailer.MailKit/)|
|NullDesk.Extensions.Mailer.SendGrid                                                        |[![MyGet](https://img.shields.io/myget/nulldesk-ci/vpre/NullDesk.Extensions.Mailer.SendGrid.svg)](https://www.myget.org/feed/nulldesk-ci/package/nuget/NullDesk.Extensions.Mailer.SendGrid)|[![NuGet](https://img.shields.io/nuget/v/NullDesk.Extensions.Mailer.SendGrid.svg)](https://www.nuget.org/packages/NullDesk.Extensions.Mailer.SendGrid/)|
|NullDesk.Extensions.Mailer.History.EntityFramework                                         |[![MyGet](https://img.shields.io/myget/nulldesk-ci/vpre/NullDesk.Extensions.Mailer.History.EntityFramework.svg)](https://www.myget.org/feed/nulldesk-ci/package/nuget/NullDesk.Extensions.Mailer.History.EntityFramework)|[![NuGet](https://img.shields.io/nuget/v/NullDesk.Extensions.Mailer.History.EntityFramework.svg)](https://www.nuget.org/packages/NullDesk.Extensions.Mailer.History.EntityFramework/)|
|NullDesk.Extensions.Mailer.History.EntityFramework.SqlServer                               |[![MyGet](https://img.shields.io/myget/nulldesk-ci/vpre/NullDesk.Extensions.Mailer.History.EntityFramework.SqlServer.svg)](https://www.myget.org/feed/nulldesk-ci/package/nuget/NullDesk.Extensions.Mailer.History.EntityFramework.SqlServer)|[![NuGet](https://img.shields.io/nuget/v/NullDesk.Extensions.Mailer.History.EntityFramework.SqlServer.svg)](https://www.nuget.org/packages/NullDesk.Extensions.Mailer.History.EntityFramework.SqlServer/)|

## Contents

- [Features](#features)
- [Package Descriptions](#pacakge-descriptions)
- [Basic Usage](#basic-usage)
  - [Mailer Instantiation](#mailer-instantitaion)
  - [Mailer Factory Usage](#mailer-factory-usage)
  - [Dependency Injection](#dependency-injection)
  - [Logging with ILogger](#ilogger)
  - [Delivery History with IHistoryStore](#ihistorystore)
- [Creating MailerMessages](#creating-messages)
  - [Fluent Message Builder](#message-builder)
  - [Fluent Extensions](#fluent-extensions)
  - [Class Instantiation](#class-instantiation)
  - [Substitutions and PersonalizedSubstitutions](#subs)
  - [Body Content](#bodycontent)
  - [Templates](#tempaltes)
  - [Attachments](#attachments)
- [Creating your own Mailer](#custom-mailer)

## <a name="features"></a>Features

- Template support for all mailers
  - local filesystem body templates for mail services without their own
  - native templates for mail services that support them
- Replacement variables (substitutions) for message subject and content
  - works with explicitly defined message bodies, as well as templates
  - support message-wide and per-recipient variables both     
- Optional Fluent Message Builder API 
- Cross platform Netstandard 1.3 packages
  - Compatible with classic .Net and .Net core applicaitons.
- Uses MS Options extensions for simplified and flexible configuration  
- Optional supports for Microsoft logging extensions 
- Use with or without DI frameworks
  - Build-in extensions for Microsoft's DI extensions
  - Factory for cases where DI isn't desired or needed  
- Sample applications demonstrating common scenarios
- Editor support for Visual Studio Code and Visual Studio 2017
- Optional message and delivery history store

## <a name="pacakge-descriptions"></a>Package Descriptions

|                                                                                  |           |
|----------------------------------------------------------------------------------|-----------|
|NullDesk.Extensions.Mailer                                  |Base classes and interfaces for the mailer extensions, and settings.|
|NullDesk.Extensions.Mailer.MailKit                          |SMTP Relay Email service using the popular cross platform [MailKit library](https://github.com/jstedfast/MailKit). Includes support for basic Email Template files.|
|NullDesk.Extensions.Mailer.SendGrid                         |SendGrid email service using SendGrid APIs. Supports basic usage of SendGrid templates; can be inherited for more advanced SendGrid functionality.|
|NullDesk.Extensions.Mailer.History.EntityFramework          |Base classes for message and delivery history using entity framework |
|NullDesk.Extensions.Mailer.History.EntityFramework.SqlServer|Implemenation of message and delivery history using entity framework targeting for MS SQL Server |
|_NullDesk.Extensions.Mailer.NetMail_                         |*(coming soon)* SMTP Relay Email service using the cross-platform System.Net.Mail framework from Microsoft.|


## <a name="basic-usage"></a>Basic Usage

Usage of the mailer extensions generally follows the following steps:

- Obtain a mailer instance (from DI, the factory, or by instantiating one yourself)
- Add one or more Messages to the Mailer instance. 
  - You can add as many messages as you want
    - Use the AddMessage method, or call CreateMessage to use the fluent message buidler API.
  - When a message is added to the mailer, it is converted into one or more DeliveryItem instances
    - The mailer instance keeps track of these before and after delivery
    - One DelieryItem instance is create for each message and recipient
    - If using the history store these will be recorded individually
- Once you have added all the messages you wish to deliery to the Mailer, simply call the SendAllAsync method. 

> The built-in Mailers are reusable by default, but it is recommended to create a new instance each time instead. Since the mailer will continue to track previously delivered items in memory, disposing of the mailer instance after each message (or batch) is preferrable. 

### <a name="mailer-instantitaion"></a>Mailer Instantiation

The simplest usage is to just instantiate the mailer of your choice, use the fluent message builder API to create a message, then send it:

         var settings = new SendGridMailerSettings
         {
             ApiKey = "123",
             FromDisplayName = "Person Name",
             FromEmailAddress = "someone@toast.com",
             IsSandboxMode = false
         };
         using (var mailer = new SendGridMailer(new OptionsWrapper<SendGridMailerSettings>(settings)))
         {
             mailer.CreateMessage(b => b
                .Subject("Message for %name%")
                .And.To("recipient@toast.com").WithDisplayName("Recipient Name")
                .And.ForBody()
                    .WithHtml(htmlContentString)
                    .AndPlainText(textContentString)
                .And.WithSubstitutions(replacementVariablesDictionary)
                .And.WithAttachments(attachmentFileNamesList)
                .Build());

             var deliveryItem = await mailer.SendAllAsync(CancellationToken.None);
             if(deliveryItem.IsSuccess)
             {
                 //happy dance!
             }
         }

In the above example, the settings are supplied via the <code>OptionsWrapper&lt;T&gt;</code> class; but if your application is using the Microsoft Options Extensions for configuration, you can use IOptions, IOptionsSnapshot, etc. for more advanced control of runtime settings.

### <a name="mailer-factory-usage"></a>Mailer Factory Usage

For most real-world scenarios, you would use dependency injection or the provided factory to obtain mailer instanaces, without having to supply all the constructor parameters each time.

Using the mailer factory, you can configure the mailer once in startup:

        var settings = new SendGridMailerSettings
        {
            ApiKey = "123",
            FromDisplayName = "Person Name",
            FromEmailAddress = "someone@toast.com",
            IsSandboxMode = false
        };

        var factory = new MailerFactory();
        factory.AddSendGridMailer(sendGridSettings); 

Then anytime you need to send mail, just grab a new mailer instance from the factory and go:

        using(var mailer = factory.GetMailer())
        {
            mailer.CreateMessage(b => b
                .Subject("Message for %name%")
                .And.To("recipient@toast.com")
                    .WithDisplayName("Recipient Name")
                    .WithPersonalizedSubstitution("%name%", "Mr. Toast")
                .And.To("someoneElse@toast.com")
                    .WithDisplayName("Other Recipient Name")
                    .WithPersonalizedSubstitution("%name%", "Someone Else")
                .And.ForTemplate(myTemplateName)
                .Build());

            var deliveryItem = await mailer.SendAllAsync(CancellationToken.None);
            if(deliveryItem.IsSuccess)
            {
                //happy dance!
            }
        }

### <a name="dependency-injection"></a>Dependency Injection

When using dependency injection frameworks, it is best to register the mailers to be transient instances. This example demonstrates this using Microsoft's own dependency injection extensions:

        public IServiceProvider ConfigureServices(IServiceCollection services)
        {
            services.AddLogging();
            services.AddOptions();
            services.Configure<SendGridMailerSettings>(s =>
            {
                    s.ApiKey = "123";
                    s.FromDisplayName = "Person Name";
                    s.FromEmailAddress = "someone@toast.com";
                    s.IsSandboxMode = false;
            });

            services.AddTransient<IMailer, SendGridMailer>();

            return services.BuildServiceProvider();
        }

### <a name="ilogger"></a>(optional) Logging with ILogger

The mailer extensions support logging via the ILogger interface from [Microsoft's Logging Extensions](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging). While designed for ASP.NET Core, the logging extensions can be used in any .Net Core project and Full-Framework .Net projects version 4.5 and higher. Additionally, the Logging extensions can be configured to interoperate with almost all popular logging frameworks for .Net --log4net, serilog, etc. 

All classes derived from the <code>Mailer</code> class take an optional <code>ILogger</code> or <code>ILogger&lt;T&gt;</code> constructor parameter, as do the extensions for registering the Mailer with the MailerFactory.

Example using the MailerFactory:

        var loggerFactory = new LoggerFactory();
        //configure your desired logging providers
        loggerFactory.AddConsole(consoleLoggerConfig);

        var mailerFactory = new MailerFactory();
        mailerFactory.AddSendGridMailer(
            sendGridSettings,
            loggerFactory.CreateLogger<SendGridMailer>());

Example when using MS DI extensions:

        public IServiceProvider ConfigureServices(IServiceCollection services)
        {
            services.AddLogging(); //that's all you need

            services.AddOptions();
            services.Configure<SendGridMailerSettings>(s =>
            {
                    s.ApiKey = "123";
                    s.FromDisplayName = "Person Name";
                    s.FromEmailAddress = "someone@toast.com";
                    s.IsSandboxMode = false;
            });

            services.AddTransient<IMailer, SendGridMailer>();

            return services.BuildServiceProvider();
        }


Example when instantiating a Mailer directly:

        var loggerFactory = new LoggerFactory();

        //configure your desired logging providers
        loggerFactory.AddConsole(consoleLoggerConfig);

        using (var mailer = new SendGridMailer(
            new OptionsWrapper<SendGridMailerSettings>(settings),
            loggerFactory.CreateLogger<SendGridMailer>()))
        {
            //stuff
        }

If a logger isn't supplied, the framework will automatically use the <code>Microsoft.Extensions.Logging.Abstractions.NullLogger</code> instance.


### <a name="ihistorystore"></a>(optional) Delivery History with IHistoryStore

The mailer extensions also support recording mesages and their delivery details in an optional message history store. 

This works very similar to the logging support.

Example using the MailerFactory:

        var loggerFactory = new LoggerFactory();
        //configure your desired logging providers
        loggerFactory.AddConsole(consoleLoggerConfig);

        var builder = new DbContextOptionsBuilder<SqlHistoryContext>()
                        .UseSqlServer(connectionString);

        var mailerFactory = new MailerFactory();
        mailerFactory.AddSendGridMailer(
            sendGridSettings,
            loggerFactory.CreateLogger<SendGridMailer>(),
            new EntityHistoryStore<SqlHistoryContext>(builder.Options));

Example when using MS DI extensions:

        public IServiceProvider ConfigureServices(IServiceCollection services)
        {
            services.AddLogging();

            services.AddOptions();

            services.AddSingleton<DbContextOptions>(s =>
            {
                var builder = new DbContextOptionsBuilder<SqlHistoryContext>()
                    .UseSqlServer(connectionString);
                return builder.Options;
            });
            services.AddSingleton<IHistoryStore, EntityHistoryStore<SqlHistoryContext>>();

            services.Configure<SendGridMailerSettings>(s =>
            {
                    s.ApiKey = "123";
                    s.FromDisplayName = "Person Name";
                    s.FromEmailAddress = "someone@toast.com";
                    s.IsSandboxMode = false;
            });

            services.AddTransient<IMailer, SendGridMailer>();

            return services.BuildServiceProvider();
        }

Example when instantiating a Mailer directly:

        var loggerFactory = new LoggerFactory();

        //configure your desired logging providers
        loggerFactory.AddConsole(consoleLoggerConfig);

        var builder = new DbContextOptionsBuilder<SqlHistoryContext>()
                        .UseSqlServer(connectionString);


        using (var mailer = new SendGridMailer(
            new OptionsWrapper<SendGridMailerSettings>(settings),
            loggerFactory.CreateLogger<SendGridMailer>(),
            new EntityHistoryStore<SqlHistoryContext>(builder.Options)))
        {
            //stuff
        }

When using the provided EF hsitory store, it is up to your if your client application will manage the database by calling EF migrations in code, of if you want to handle running migrations as part of your deployment process. If you want to do this in code, simple run the migration during your application's initilization.

        var builder = new DbContextOptionsBuilder<SqlHistoryContext>()
                        .UseSqlServer(connectionString);

        using (var ctx = new SqlHistoryContext(builder.options))
        {
            ctx.Database.Migrate();
        }

## <a name="creating-messages"></a>Creating MailerMessages

### <a name="message-builder"></a>Fluent Message Builder

### <a name="fluent-extensions"></a>Fluent Extensions

### <a name="class-instantiation"></a>Class Instantiation

### <a name="subs"></a>Substitutions and PersonalizedSubstitutions

### <a name="bodycontent"></a>Body Content

### <a name="templates"></a>Templates

### <a name="attachments"></a>Attachments

## <a name="custom-mailer"></a>Creating your own mailers

To implement your own mailer, simply implement a class that inherits <code>Mailer&lt;TSettings&gt;</code>.

For the above interfaces, <code>TSettings</code> is a custom class that implements <code>IMailerSettings</code>, then adds any custom configuration properties your mailer or the underlying mail framework needs.

        public class MyMailerSettings : IMailerSettings
        {
            public string FromEmailAddress { get; set; }

            public string FromDisplayName { get; set; }

            //... any other properties your mailer needs
        }

Every effort has been made to keep inheritance of <code>Mailer&lt;TSettings&gt;</code> fairly straight forward.

The primary method you must supply is a method that can deliver a single <code>DeliveryItem</code> using the email service your mailer supports. The method should return the DeliveryItem if it was successfully sent, or throw an exception if something went wrong. The base class will handle logging the exception, updating the DeliveryItem's properties, recording the delivery attempt to the history store, etc. All you have to do is send the email via the mail service, and either throw an exception or return the deliery item if it was sent.

        public class MySimpleMailer : Mailer<MyMailerSettings>
        {
            protected override async Task<DeliveryItem> DeliverMessageAsync(
                DeliveryItem deliveryItem,
                CancellationToken token = default(CancellationToken))
            {
                //... implementation here
            }
        }


