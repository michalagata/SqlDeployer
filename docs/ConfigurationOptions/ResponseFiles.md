---
title: "Response (.rsp) Files"
permalink: /response-files/
parent: Configuration options
---
# Response (.rsp) Files

As sqldep is built on the `System.CommandLine` libraries it has built in [support for Response (`.rsp`) files](https://github.com/dotnet/command-line-api/blob/main/docs/Features-overview.md#Response-files).  These can be used to pass a standard set of command line arguments to the tool in lieu of a long command line.

For example, if you created a `sqldep_settings.rsp` file with the following content
``` bash
# Custom RSP for a local development drop/create migration
-cs SomeConnectionString
-f ./db
--env DEV
--drop
--version 1.0
--ut=my_token=myvalue
```

and ran it via `sqldep @./sqldep_settings.rsp` it would be the equivalent to the following command:
``` bash
sqldep -cs SomeConnectionString -f ./db --env DEV --drop --version 1.0 --ut=my_token=myvalue
```

Note that the `@` before the file path is required.