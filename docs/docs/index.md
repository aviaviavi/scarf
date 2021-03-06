# Using Scarf as software distribution tool

## What is Scarf? Why should I use it?

Scarf makes it easy to publish your packages and developer tools to users on
Linux and Mac, understand how your software is being used, and charge the
companies that benefit from your work in a commercial setting.

Scarf is a developer-centric, cross-platform system package manager. If you
distribute your software with Scarf, your users will be able to easily install
it with the `scarf` CLI, and you'll gain insights into how your software is used
such as:

- Install counts
- Exit codes when your program is invoked
- Execution times
- Sub-commands and flags that are passed on the command line
- And much more

For your users who wish to use your package without reporting usage statistics
(or those who just want to support your work), Scarf gives you an easy way to
collect payments in exchange. You can easily create a Scarf-connected Stripe
account, set your desired price for your package, and your users can easily pay
to use your package without reporting usage statistics.

The best part is that you can get all of these features without writing any
code! You simply upload your package to Scarf. The end-user will install your
package via `scarf` which installs your program in a way that that Scarf can
automatically capture your package's usage statistics and enforce permission
levels.

## How it works

When a user installs your package through Scarf, it is installed inside of a
thin wrapper. This wrapper will log usage statistics automatically when it
invokes your package, and can also perform other checks, like whether the user
has purchased a particular tier of your package and its dependencies.

To get more information on how the CLI works, you can check out the code on
[Github](https://github.com/aviaviavi/scarf)

## Installing Scarf

On Debian-based linux systems, there is a dependency on `netbase`

```bash
$ sudo apt install netbase
```

To get Scarf, simply run:

```bash
$ curl -L https://scarf.sh/install | bash
```

You'll then need to add `$HOME/.scarf/bin` to the front of your `PATH`


## Creating your Scarf account

Head over to [https://scarf.sh](https://scarf.sh) to register your developer
account.

## Creating your first package

Once you're registered, you'll want to create your package on the "New Package"
page of the [Scarf website](https://scarf.sh/#/create-package). The package name
here will be what users type when they install your package, but isn't
necessarily the name of the executable they will invoke.Scarf currently supports
packages in the form of:

- A locally built archive with an executable that can be run directly on the target platform.
- An npm package that you upload to scarf rather than npm itself.

Scarf is actively adding other package installation types. If your package type
isn't supported yet, let us know what you need and we'll prioritize it!

### Define your package specification

You can now add releases to your package that your users can install! A Scarf
package release primarily involves writing a small package specification or
uploading your npm package directly.

#### Yaml specifications (archive based packages)

Standard archive based packages can be described in yaml.

```yaml
name:  "curl-runnings"
author: "Avi Press"
copyright: "2019 Avi Press"
license: "MIT"
version: "0.11.0"
# For each platform (currently MacOS and Linux_x86_64) you're distributing your release to, include an entry in distributions.
distributions:
  -
    platform: MacOS
    # The binaries your package provides. If your binary names are the same as
    # their path within your archive, you can simply provide a string of binary names
    bins:
      curl-runnings: path/inside/your/archive/curl-runnings
    # uri can be a remote or local tar archive
    uri: "https://github.com/aviaviavi/curl-runnings/releases/download/0.11.0/curl-runnings-0.11.0-mac.tar.gz"
    # [Optional] if your archive has extra files that should be included, list them here
    includes:
      - "./directories"
      - "./or-files.txt"
  -
    platform: Linux_x86_64
    bins:
      - curl-runnings
    uri: "./path/to/local/archive.tar.gz"
    includes: []
    depends:
      - scarf-packages
      - your-package
      - depends-on
```

Some notes:

- You can use `scarf check-package ./path/to/your/package-file.yaml` to
validate your package file. Currently, it won't do things like check your
archive or test your release, but it will make sure you spec type-checks, and
that you have a valid license type and platform.

- Dependency handling is still immature. Chances are, the package you depend on
isn't on Scarf yet! We're working hard to fix that, however. Just send an email
to help@scarf.sh or reach out on [gitter](https://gitter.im/scarfsh/community)
and we'll help get the packages you need. Currently any dependencies you add in
your scarf.yaml will use the latest validated version.

#### NPM

You can upload an npm based package to scarf rather than npm itself. It will be
globally installed by scarf just like any other scarf package. You'll need to
make sure your `package.json` includes a `main` entry that points to the script
that will be ultimately invoked, and the package must have a license.

You can use `scarf check-package ./path/to/your/package.json to
validate your package file.

#### Piggybacking on other package managers

If your package is already distributed with another package manager, you can
configure Scarf to invoke that package install while still getting all the other
benefits Scarf provides. This can be useful to get your package on Scarf if
Scarf doesn't yet support your exact package type natively (if this is the case,
send us an email with what you need!). Scarf will call out to the external
manager on the users system.


```yaml
...
bins: &bins
  - bin1
  - bin2
  ...
  - binN
distributions:
  -
    external: Debian
    bins: *bins
  -
    external: RPM
    bins: *bins
  -
    external: Homebrew
    installCommand: "install --special --flags my_package"
    bins: *bins
  ...

```

Currently supported third party package types include:

- Homebrew
- Debian (`apt`)
- RPM
- CPAN

Let us know if you need something else!

### Uploading your release

Once you have a valid spec, it's time to upload! You'll need your
`SCARF_API_TOKEN`, which you can find by going to your [account
page](https://scarf.sh/#/user-account). To upload, run:

```bash
SCARF_API_TOKEN=${your_token} scarf upload ./path/to/your/validated-spec.(yaml|json)
```

**Packages on Scarf can't be deleted once they're uploaded!**

Once your release is uploaded, your users can install your package with a simple:

```bash
scarf install ${your_package_name}
```

## Viewing your package analytics

Once you've pushed a release to your package, you can head over to the
[dashboard](https://scarf.sh/#/home) to see your packages install and usage
stats!

## Connecting a Stripe Account

Navigate to your account details page by clicking your username in the nav bar.
Find the `Connect to Stripe` button at the bottom, and follow Stripe's
onboarding process. It only takes a few minutes. You'll be redirected back to
Scarf when the enrollment is complete.

## Adding a private usage tier to your package

Now that you've connected a Stripe account, you're ready to start collecting
payments for your package! Navigate to your package detail page on Scarf and
click `Start monetizing <package>`. Set your price. That's it! Scarf handles all
user registration and payments so you are now fully ready to start making money
from your package. Stripe will send your payouts daily to your connected bank
account. (NOTE: your very first payout will go through 7 days after your account
is opened, so be patiend for your first payout.)

## Keeping Scarf up to date

A simple upgrade command is all you need to get the latest version of Scarf

```
$ scarf upgrade
```
