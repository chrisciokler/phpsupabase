# PHPSupabase

PHPSupabase is a library written in php language, which allows you to use the resources of a project created in Supabase ([supabase.io](https://supabase.io)), through integration with its Rest API.

## About Supabase

Supabase is "The Open Source Firebase Alternative". Through it, is possible to create a backend in less than 2 minutes. Start your project with a Postgres Database, Authentication, instant APIs, realtime subscriptions and Storage.

## PHPSupabase Features

- Create and manage users of a Supabase project
- Manage user authentication (with email/password, magic links, among others)
- Insert, Update, Delete and Fetch data in Postgres Database (by Supabase project Rest API)
- A QueryBuilder class to filter project data in uncomplicated way

## Instalation & loading

PHPSupabase is available on [Packagist](https://packagist.org/packages/rafaelwendel/phpsupabase), and instalation via [Composer](https://getcomposer.org) is the recommended way to install it. Add the follow line to your `composer.json` file:

```json
"rafaelwendel/phpsupabase" : "1.0"
```

or run

```sh
composer require rafaelwendel/phpsupabase
```

## How to use

To use the PHPSupabse library you must have an account and a project created in the Supabase panel. In the project settings (API section), you should note down your project's `API key` and `URL`. (NOTE: Basically we have 2 suffixes to use with the url: `/rest/v1` & `/auth/v1`)

To start, let's instantiate the `Service()` class. We must pass the `API key` and `url` (with one of the suffixes mentioned above) in the constructor

```php
<?php

require "vendor/autoload.php";

$service = new PHPSupabase\Service(
    "YOUR_API_KEY", 
    "https://aaabbbccc.supabase.co/auth/v1/"
);
```

The `Service` class abstracts the actions with the project's API and also provides the instances of the other classes (`Auth`, `Database` and `QueryBuilder`).

### Auth class

Let's instantiate an object of the `Auth` class

```php

$auth = $service->createAuth();
```

The `$auth` object has several methods for managing project users. Through it, it is possible, for example, to create new users or even validate the sign in of an existing user.

#### Create a new user with email and password

See how to create a new user with `email` and `password`.

```php

$auth = $service->createAuth();

try{
    $auth->createUserWithEmailAndPassword('newuser@email.com', 'NewUserPassword');
    $data = $auth->data(); // get the returned data generated by request
    echo 'User has been created! A confirmation link has been sent to the '. $data->email;
}
catch(Exception $e){
    echo $auth->getError();
}
```

This newly created user is now in the project's user table and can be seen in the "Authentication" section of the Supabase panel. To be enabled, the user must access the confirmation link sent to the email.

#### Sign in with email and password

Now let's see how to authenticate a user. The Authentication request returns a `acess_token` (Bearer Token) that can be used later for other actions and also checks expiration time. In addition, other information such as `refresh_token` and user data are also returned. Invalid login credentials result in throwing a new exception

```php

$auth = $service->createAuth();

try{
    $auth->signInUserWithEmailAndPassword('user@email.com', 'UserPassword');
    $data = $auth->data(); // get the returned data generated by request

    if(isset($data->access_token)){
        $userData = $data->user; //get the user data
        echo 'Login successfully for user ' . $userData->email;

        //save the $data->acess_token in Session, Cookie or other for future requests.
    }
}
catch(Exception $e){
    echo $auth->getError();
}
```

#### Get the data of the logged in user

To get the user data, you need to have the `access_token` (Bearer Token), which was returned in the login action.

```php

$auth = $service->createAuth();
$bearerToken = 'THE_ACCESS_TOKEN';

try{
    $data = $auth->getUser($bearerToken);
    print_r($data); // show all user data returned
}
catch(Exception $e){
    echo $auth->getError();
}
```

#### Update user data

It is possible to update user data (such as email and password) and also create/update `metadata`, which are additional data that we can create (such as `first_name`, `last_name`, `instagram_account` or any other).

The `updateUser` method must take the `bearerToken` as argument. In addition to it, we have three more optional parameters, which are: `email`, `password` and `data` (array). If you don't want to change some of this data, just set it to `null`.

An example of how to save/update two new meta data (`first_name` and `last_name`) for the user.

```php

$auth = $service->createAuth();
$bearerToken = 'THE_ACCESS_TOKEN';

$newUserMetaData = [
    'first_name' => 'Michael',
    'last_name'  => 'Jordan'
];

try{
    //the parameters 2 (email) and 3(password) are null because this data will not be changed
    $data = $auth->updateUser($bearerToken, null, null, $newUserMetaData);
    print_r($data); // show all user data returned
}
catch(Exception $e){
    echo $auth->getError();
}
```
Note that in the array returned now, the keys `first_name` and `last_name` were added to `user_metadata`.