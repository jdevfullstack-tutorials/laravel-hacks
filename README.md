[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Fjdevstatic%2Feverything-laravel&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=PAGE+VIEWS&edge_flat=false)](https://hits.seeyoufarm.com)

# Everything Laravel
Disclaimer : this is not a full Laravel tutorial.
Rather, this is a collection of my most encountered
situations, solutions and approaches. I'm
documenting it because it's really hard to
memorize these things even if you do it 
every single day and when returning
to Laravel after a long break.

TOC
* [Initial Setup](#initial-setup)
   * [Server Installation](#server-installation)
   * [Cloning the Project](#cloning-the-project)
   * [.env Config](#env-config)
   * [Composer & Artisan](#composer--artisan)
   * [Installing & Running the Project](#installing--running-the-project)
* [Ignore The Platform Flag](#ignore-the-platform-flag)
* [Unit & Integration Testing](#unit--integration-testing)
   * [Database Testing](#database-testing)
   * [Why Unit Tests ?](#why-unit-tests-)
* [Safely Deleting A Module](#safely-deleting-a-module)
* [New Routes In Backend](#new-routes-in-backend)
* [New Package Installation](#new-package-installation)
* [Adding New Column/s To A Table](#adding-new-columns-to-a-table)
* [Database Interaction](#database-interaction)
   * [Eloquent & Laravel Query Builder](#eloquent--laravel-query-builder)
   * [Joining Tables](#joining-tables)
   * [Surrogate Key or Natural Key](#surrogate-key-or-natural-key)
   * [Database Normalization & Denormalization](#database-normalization--denormalization)
* [Deploying Laravel On Test / Production Servers](#deploying-laravel-on-test--production-servers)
   * [Amazon EC2](#amazon-ec2)
   * [WinSCP & PuTTY](#winscp--putty)
   * [Git Pull or Manual File Transfer](#git-pull-or-manual-file-transfer)
   * [Different ENVs](#different-envs)
   * [Continuously Run Laravel On Test / Prod Env](#continuously-run-laravel-on-test--prod-env)


## Initial Setup
### Server Installation

Laravel has its own server when you run

`php artisan serve`

but that's not complete particularly when
you need a database like MySQL, so
we need to install a development server.

On Windows OS for the local development environment, 
we need XAMPP. Go ahead and install it.

### Cloning the Project

Most common way is to `git clone`.

If private, once you have access to the repo,
you can clone it. If public, just clone it.

And make sure that it is inside the `htdocs`
folder of XAMPP.

Another option is to simply download the
project from GitHub. But you don't
get the history of the project.

### .env Config
Create a new text file name 
`.env` inside the root of the project.

Ask the concerned person for the 
exact content of the `.env` file if this is private.

If public, mostly it's the `example.env`, so go ahead
copy the content of it and paste it on the actual `.env`.

Ever wonder why we don't include `.env` on GitHub ? 

Because it tends to be different on different
machines and usually contains sensitive data, like
the API token.

### Composer & Artisan
Familiarity with the Composer and Artisan is needed.

`composer` - is a dependency management tool for PHP

`artisan` - this is the command line utility of Laravel

more on this here,

https://laravel.com/docs/10.x/installation

### Installing & Running the Project

After cloning a project,

1. `composer install`

2. `php artisan migrate:fresh --seed`

when it's API and you have implemented
this 

https://dev.to/grantholle/implementing-laravel-s-built-in-token-authentication-48cf

you need to run 

3. `php artisan make:token`

   copy paste the token on the frontend or any other
   service that will be needing it

then finally, the moment of truth,

4. `php artisan serve`

and there you have it, enjoy coding !

## Ignore The Platform Flag
If you run the `composer install` command and you get an error,
temporarily we can use `--ignore-platform-reqs` to bypass the error 
but it's not a good practice. You need to identify 
what is causing the error.

For example, when we use an external 
package QR Code, we need to enable GD Extension, hence
edit `php.ini` and uncomment `extension=gd`. Then
try `composer install` again.

This way we are installing the same versions
of the required packages.

## Unit & Integration Testing

In modular approach, you need to tweak the default
TestCase and the path so all modules will be tested,

then,

1. make sure that the `api_token` is updated 
on the `.env` file, it's needed for the 
access of the endpoints, else all test will simply
fail, if there is such authentication

2. `php artisan config:cache`

3. `php artisan test`

### Database Testing
This is very critical as all our tests, particularly
backend API will always have the database involved. 

In my case, because I followed the generic
authentication and that token needs to be pasted
so we can call the endpoints, I cannot use
`Database Migrations` and `RefreshDatabase`, rather
what fits in my situation is Database Transactions

https://laravel.com/docs/5.4/database-testing#using-transactions

### Why Unit Tests ? 
1. writing Unit Tests makes you realize that
each function should do a task that can pass
or fail by testing it

2. allows you to make functions decoupled as
much as possible

3. it's a kind of regression testing where when
you add new features to an existing system,
you still are at least confident that
you are not breaking the codebase by just
running the unit tests and still passing them

As for me, the third reason is what I really like
the most, because that happens all the time,
you just modify or add some new feature then
you realize later on, other features are broken
because of that.

## Safely Deleting A Module
using `nwidart/laravel-modules`, 
deleting a module should be handled correctly

1. `php artisan module:disable your_module_name`
2. `php artisan cache:clear`
3. delete module directory manually
4. `php artisan cache:clear`

other developers that will pull these changes
will simply pull it and run

`php artisan cache:clear`

## New Routes In Backend
after pulling and when there are new routes in the backend,

`php artisan route:cache`

## New Package Installation
When a developer installed a new package locally,
the `composer.json` & `composer.lock` will be updated
and this should be pushed to GitHub `main` / `master branch`

This way, other developers will not install it 
the way it was installed like 

`composer require ...`

and we avoid merge conflicts in config files
and to keep the same package versions.

Other developers will just run 

`composer install`

after pulling from GitHub `main` / `master` branch.

Depending on the package, like if there
are modified files in the `config` directory,
we need to run

`php artisan config:cache`

or 

`php artisan cache:clear`

for the changes to take effect.

## Adding New Column/s To A Table
If not yet having the actual data, developers have
the freedom to simply add new columns in the original
migration file, 

but this needs to be migrated as fresh

`php artisan migrate:fresh` 

but we will be losing the data but
since it's not actual data, this is
acceptable.

Otherwise having the actual data on production server,
we must always create a new migration file 

`php artisan module:make-migration <add_new_column_to_table> ModuleName`

and simply run

`php artisan migrate`

when pushed on GitHub `master` / `main`, other developers will
simply pull it and migrate too

and there are two conditions here:

1. set column to nullable - updating the content
can have problems in the future particularly
if there are existing data on the table, so
default null to be safe

2. column cannot be unique - if it's null and
there are existing rows that are null also 
then it's not unique anymore 

after creating or editing the migration file,

1. we need to update the validation 
and sanitation code block

2. then finally, the Store & Update methods of the Controller

3. unit tests for the new column/s and 
functionality should be added too

## Database Interaction

### Eloquent & Laravel Query Builder
In Laravel, it's best to use the `Laravel Query Builder`
and the `Eloquent ORM`. Why ? 

First is because of security, 
namely SQL injection.

Second, this unifies the syntax whether, for example,
you are using MySQL or PostgreSQL.

### Joining Tables
I usually use `LEFT JOIN` in my queries. You can
left join more than two tables. Related to
this is the choice whether you use Natural Key
or Surrogate Key.

### Surrogate Key or Natural Key
There is a tendency to use Natural Key as it has
meaning and is straightforward. Also, you can
easily implement Searching / Filtering using this.

But if the Natural Key has the tendency to
be changed, then it's not ideal. It's a headache
actually. So, for the very first time, make sure
whether a Natural Key will be 
subjected to change or not.

### Database Normalization & Denormalization
If you will not be reaching 100k records or more
then it's sufficient to have up to 3rd Normal Form, 
otherwise, database structure will be too complex.
Once reaching 3rd Normal Form, it's now time
to denormalize to lessen the number of tables.

Take note also that using purely Surrogate Key
violates this, particularly the 3rd Normal Form.
It has no relation to the actual table data.

## Deploying Laravel on Test / Production Servers
There are so many ways to deploy Laravel, others even
have the exact configuration for Laravel only.

### Amazon EC2 
But as for me, the best is Amazon EC2 because the
way you install it remotely is like the way you
install it locally, the only difference is that
mostly it's using Ubuntu whether it uses `Apache` or
`nginx` servers. So familiarity with Linux is needed. 

### WinSCP & PuTTY
To access
it remotely, we need WinSCP and it has PuTTY, 
an open-source terminal emulator. 

If you know FTP, you will not find it difficult
to use WinSCP, as it is an SFTP and FTP client for Windows.
But not usually to transfer files particularly the code.
For code, use `git pull` to pull changes from GitHub.

For modifying server settings, because we are using
an FTP client, we can modify without
using the default for Linux, like `sudo nano ...` but
rather opening files much like we open it on Notepad. 

### Git Pull or Manual File Transfer
Make sure that the files you are modifying are outside
your project. If that is being tracked by Git and you
changed it, you can end up with merge conflicts. This will
cause confusion also. 

For all Git tracked files,
modify it first in local then push to GitHub then pull to
your VM. If that is VM specific code and your local will
be affected, you can enclose it with a conditional, like
it will check whether the `env` is production / test / local.

### Different ENVs
Ever wonder why we have different environments ? Why, for
example, we cannot use on our local a production env or test
env ? Because it serves differently. For example, in local
we can simply run `composer update` without being bothered much.
That's not the case in production setting. So you need a local
env to try things out.

As for the Test env, usually this has to do with testing.
There is the test server that is usually being accessed by
the QA team. It's like a production setup but the `debug` mode
is on.

And finally the production environment, which does not have
the debug mode. Imagine returning the details of the server
and the errors / bugs. Well, you are giving the
hackers the idea of the most vulnerable part of your system.

You don't update the production environment too, as this
can ruin the entire production env. Imagine if you have
version 1, and 2-3 functionalities of this are deprecated
but your code is still using those functionalities then
you upgraded to version 2, then
boom ... you just ruin your project.

### Continuously Run Laravel On Test / Prod Env
When in Test / Production environments, we
need to run the application not with 

`php artisan serve`

but rather using `pm2`

https://pm2.io/docs/runtime/guide/installation/

and follow this

https://awangt.medium.com/run-and-monitor-laravel-queue-using-pm2-dc4924372e03

it can be alongside the frontend if that is separate but
on the same VM.
