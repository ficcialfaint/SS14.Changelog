# SS14.Changelog

Automatic changelog service for generation changelogs for Space Station 14. This runs as an old-style GitHub [webhook](https://docs.github.com/en/developers/webhooks-and-events/webhooks/about-webhooks): you will need a server to host this on and to set up a webhook on your repo.

## Setup

First, compile for whatever you're going to host this on:

```
dotnet publish -c Release --no-self-contained -r linux-x64
```

Output files will be in `SS14.Changelog/bin/Release/net5.0/linux-x64/publish/` or something like that. Put them on your server somewhere. Have it run as a daemon, and expose its port to the internet via a reverse proxy or something, and wire stuff up on GitHub.

For reference, this is what the file layout on our server looks like:

```
/opt/ss14_changelog
├── appsettings.yml
├── bin
│   ├── SS14.Changelog
│   ├── SS14.Changelog.dll
│   └── <rest of publish files generated above>
├── run.sh # Just a wrapper bash script to run bin/SS14.Changelog, nothing fancy.
└── ssh_key
```

Here is what you'll want to put in `appsettings.yml`. Please read every comment:

```yml
Serilog:
  Using: [ "Serilog.Sinks.Console", "Serilog.Sinks.Loki" ]
  MinimumLevel:
    Default: Information
    Override:
      SS14: Verbose
      Microsoft: "Warning"
      Microsoft.Hosting.Lifetime: "Information"
      Microsoft.AspNetCore: Warning

  WriteTo:
    - Name: Console
      Args:
        OutputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3} {SourceContext}] {Message:lj}{NewLine}{Exception}"

  Enrich: [ "FromLogContext" ]

  # If you want Loki logging.
  #Loki:
  #  Address: "http://localhost:3101"
  #  Name: "centcomm"

# Change URL/port to bind to here.
urls: "http://localhost:45896"

Changelog:
  # Secret configured in the github webhook, to ensure authenticity.
  GitHubSecret: "<SECRET>"
  # The branch to look at for generating changelogs.
  ChangelogBranchName: "master"
  # The SSH Key to use to push/pull changes.
  SshKey: '/opt/ss14_changelog/ssh_key'
  # The on-disk repo to keep up to date and to generate changelogs in.
  # You need to initialize this manually with git clone.
  ChangelogRepo: '/var/lib/changelog/repo/'
  # How long to wait after a changelog-change has been merged/pushed before we generate changelogs and push a commit.
  DelaySeconds: 60

AllowedHosts: "*"
```

Uhhh, that should be everything, I think?
