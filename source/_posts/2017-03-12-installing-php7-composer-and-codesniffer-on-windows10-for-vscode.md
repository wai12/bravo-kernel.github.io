title: Installing PHP7, Composer and CodeSniffer on Windows 10 (for VSCode)
date: 2017-03-12 18:00:00
tags:
  - sysop
  - cakephp
  - vscode
  - windows10
  - composer
  - phpcs
categories:
  - sysop
---

This post will guide you through clean (non-polluting) Windows 10 installations
of PHP 7, Composer and PHP CodeSniffer 
and will show you how to integrate it all with Visual Studio Code as a bonus.

## 1. Installing PHP

[Download](http://windows.php.net/download) one of the PHP binaries suitable for your system 
(this post will use
[the 64-bit Thread Safe version of PHP 7.1.2](http://windows.php.net/downloads/releases/php-7.1.2-nts-Win32-VC14-x64.zip)) and:

- extract the downloaded zip file
- rename the resultant directory to `php-x64`
- move the renamed directory to `"C:\Program Files"`

*FYI: may choose any other directory name but this post will assume PHP installed in `"C:\Program Files\php-x64"`*

### Enable OpenSSL

PHP will not have OpenSSL enabled by default which will cause Composer errors
later on so we might as well enable it right now by opening a
Command Prompt (with elevated admin permissions) and running:

```cmd
cd "C:\Program Files\php-x64"
cp php.ini-production php.ini
```

Close the Command Prompt and open `"C:\Program Files\php-x64\php.ini"` with
your editor to uncomment the following lines:

```ini
extension_dir = "ext"
extension=php_openssl.dll
```

## 2. Installing Composer

Download and run the Composer [Windows installer](https://getcomposer.org/download/).

Make sure that the installer detects your PHP installation before completing the installation:

<br />

{% asset_img composer-install-verify-php-detection.png 'Screenshot of Composer installation detecting PHP' %}

Good to know:

- the composer binary is found in `"C:\ProgramData\ComposerSetup\bin"`
- the `PATH` (system) environment variable has been updated to include the above bin directory

### Change the global install path

By default Composer will install **global** packages into
`C:\Users\%username%\AppData\Roaming` which can become quite cluttering
so we change it.

1. Create new directory `"C:\Program Files\php-x64\composer"`

2. Open the system control panel by pressing `WINDOWS KEY`+`PAUSE/BREAK KEY`

3. Select `Advanced system Settings`

4. Select `Environmental variables`

5. Under `System Variables`, press `New` and enter the following settings:

  <br />

  {% asset_img composer-global-install-path-env.png 'Screenshot of new system variable' %}

6. Select `OK` to round up this step

### Add vendor bin directory to PATH

To make sure your system will be able to find composer installed binaries like `phpcs`
we will need to add the global vendor directory to the system `PATH` environment
variable.

1. Open the system control panel (WINDOWS KEY +PAUSE/BREAK)

2. Select `Advanced system Settings`

3. Select `Environmental variables`

4. Under `System Variables`, select existing `Path` variable

5. Select `Edit` 

6. Select `New` and enter `C:\Program Files\php-x64\composer\vendor\bin` like shown below:

  <br />

  {% asset_img composer-global-vendor-bin-path-env.png 'Screenshot of editing existing PATH system variable' %}

7. Press `OK` trice to apply the setting

Make sure to close all open any CMD boxes or they will not
be aware of the new environment variable.

## 3. Installing PHP_CodeSniffer

Open a Command Prompt (without elevated permissions) and run:

```cmd
composer global require "squizlabs/php_codesniffer=*"
phpcs --version
```

## 4. Installing additional Coding Standards

Installation instructions will differ per additional coding standard
so we will just use [cakephp-codesniffer](https://github.com/cakephp/cakephp-codesniffer)
as an example here.

Open your global `C:\Program Files\php-x64\composer\composer.json` file and manually add
the cakephp-sniffer package so it looks like this:

```json
{
    "require": {
        "squizlabs/php_codesniffer": "*",
        "cakephp/cakephp-codesniffer": "2.*"
    }
}
```

Now open a Command Prompt (without elevated permissions) and run the following
commands to install the coding standard and make the path known to phcs:
 
```cmd
composer global update
phpcs --config-set installed_paths "C:\Program Files\php-x64\composer\vendor\cakephp\cakephp-codesniffer"
phpcs --config-show
```

There is no need to configure the default coding standard as this will be handled on a 
per-project basis inside Visual Studio Code but you are free to do so anyways by
running:

```cmd
phpcs --config-set default_standard CakePHP
```

*FYI: all Sniffer settings also in C:\Program Files\php-x64\composer\vendor\squizlabs\php_codesniffer\CodeSniffer.conf*

## 5. Reboot your computer

To be absolutely sure all programs and services are aware of the new environment
variables you should now reboot your computer. **If you are not using Visual Studio
Code** you may also skip the rest of this tutorial.

## 6. Integrating Composer with VSCode

[Ioannis Kappas](https://github.com/ikappas) has created
a very neat [VSCode composer plugin](https://marketplace.visualstudio.com/items?itemName=ikappas.composer)
that allows you to lint `composer.json` files. To set it up:

1. Open Visual Studio Code

2. Press CTRL+P

3. Paste `ext install Composer publisher:"Ioannis Kappas"`

4. Install the plugin by clicking that green button

5. Reload the window by clicking that blue button to activate the plugin

6. Open the settings window (File > Preferences > Settings) 

7. Add the following line to the `USER SETTINGS` section:

  `"composer.executablePath": "c:\\ProgramData\\ComposerSetup\\bin\\composer.bat"`


### Verify

To verify the plugin is functioning properly:

- Open a `composer.json` file in Visual Studio Code

- Press F1

- Type `validate`

- Select `Composer: Validate` and pressing enter

- If things went well you should see something like this:

  <br />

  {% asset_img vscode-composer-integration.png 'Screenshot of Composer validation feedback' %}

## 7. Integrating PHP CodeSniffer with VSCode

The [VSCode phpcs plugin](https://marketplace.visualstudio.com/items?itemName=ikappas.phpcs)
is another great add-on by [Ioannis Kappas](https://github.com/ikappas)
and allows you to check your code against various coding standards.
To set ip up:

1. Open Visual Studio Code

2. Press CTRL+P

3. Paste `ext install phpcs publisher:"Ioannis Kappas"`

4. Install the plugin by clicking that green button

5. Reload the window by clicking that blue button to activate the plugin

  *Please note that code sniffing will be active from this point on.*

### Configure

Even though you are free to use your own
[configuration strategy](https://marketplace.visualstudio.com/items?itemName=ikappas.phpcs)
we will configure the plugin to **disable code sniffing by default**,
basically requiring you to enable it on a per project/VSCode workspace basis.

To disable sniffing on the global level:

1. Open Visual Studio Code

2. Open the settings window (File > Preferences > Settings)

3. Add the following line to the `USER SETTINGS` section:

   `"phpcs.enable": false`

To enable code sniffing for your project/workspace:

1. Open Visual Studio Code

2. Open one of your projects/workspaces

3. Open the settings window (File > Preferences > Settings)

4. Add the following lines to the `WORKSPACE SETTINGS` section:

   ```json
    "phpcs.enable": true,
    "phpcs.standard": "CakePHP",
    "phpcs.ignore": "config, webroot"
   ```

   *Please note that the ignore part is optional and that non-existing directories/files will cause errors.*

### Verify

To verify the plugin is functioning properly:

1. Open one of your projects with Visual Studio Code

2. Open one of your `.php` files

3. Make a change that will trigger a sniffer violation

4. Save the file (realtime sniffing not supported yet)

5. You should see a red tilde (`~`) explaining the sniff violation when hovered:

  {% asset_img vscode-phpcs-hover-tilde.png 'Screenshot of phpcs violation when hovering tilde' %}

6. The `Problems` window (View > Problems) should display a summary of all phpcs violations:

  {% asset_img vscode-phpcs-problems-output.png 'Screenshot of VSCode Problems window listing phpcs violations' %}
