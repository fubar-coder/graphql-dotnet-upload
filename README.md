# graphql-dotnet-upload

[![Build Status](https://dev.azure.com/lassahn/graphql-dotnet-upload/_apis/build/status/JannikLassahn.graphql-dotnet-upload?branchName=master)](https://dev.azure.com/lassahn/graphql-dotnet-upload/_build/latest?definitionId=1&branchName=master)
[![NuGet](https://img.shields.io/nuget/v/GraphQL.Upload.AspNetCore.svg?style=flat)](https://www.nuget.org/packages/GraphQL.Upload.AspNetCore/)

This repository contains an experimental implementation of the [GraphQL multipart request spec](https://github.com/jaydenseric/graphql-multipart-request-spec) based on ASP.NET Core.

## Installation
You can install the latest version via [NuGet](https://www.nuget.org/packages/GraphQL.Upload.AspNetCore/).
```
PM> Install-Package GraphQL.Upload.AspNetCore
```
Preview versions from the develop branch are available via [GitHub Packages](https://github.com/JannikLassahn/graphql-dotnet-upload/packages).

## Usage

Register the middleware in your Startup.cs.

This middleware implementation **only** parses multipart requests. That's why we're using additional middleware ([graphql-dotnet/server](https://github.com/graphql-dotnet/server)) to handle other request types.
```csharp
public void ConfigureServices(IServiceCollection services)
{
  services.AddSingleton<MySchema>()
          .AddGraphQLUpload()
          .AddGraphQL();
}

public void Configure(IApplicationBuilder app)
{
  app.UseGraphQLUpload<MySchema>()
     .UseGraphQL<MySchema>();
}
```

Register a `FormFileConverter` within your MySchema.cs.
```csharp
public class MySchema : Schema
{
    public MySchema()
    {
        Query = new Query();
        Mutation = new Mutation();
        RegisterValueConverter(new FormFileConverter());
    }
}
```

Use the upload scalar in your resolvers. Files are exposed as `IFormFile`. 
```csharp
Field<StringGraphType>(
    "singleUpload",
    arguments: new QueryArguments(
        new QueryArgument<UploadGraphType> { Name = "file" }),
    resolve: context =>
    {
        var file = context.GetArgument<IFormFile>("file");
        return file.FileName;
    });
```

## Testing
Take a look at the tests and the sample and run some of the cURL requests the spec lists if you are curious.

CMD:
```shell
curl localhost:54234/graphql ^
	-F operations="{ \"query\": \"mutation ($file: Upload!) { singleUpload(file: $file) { name } }\", \"variables\": { \"file\": null } }" ^
	-F map="{ \"0\": [\"variables.file\"] }" ^
	-F 0=@a.txt
```
Bash:
```shell
curl localhost:54234/graphql \
	-F operations='{ "query": "mutation ($file: Upload!) { singleUpload(file: $file) { name } }", "variables": { "file": null } }' \
	-F map='{ "0": ["variables.file"] }' \
	-F 0=@a.txt
```
