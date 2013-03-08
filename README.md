# DSL platform Tutorial

## Introduction

DSL platform is a multi-language platform inspired by Domain driven design.

<!-- @todo uljepsati uvod  -->

In this tutorial we'll show how you can rapidly develop a PHP application by using DSL platform to generate your model files and persits... @bla NEVALJA. We' ll write a blog apllication that covers the basics. We want to 

## Setup

### What you'll need:

- PHP 5.3 or greater
- basic OOP, you should know how to write your own class, create a new object instance and call its methods, that should do it for the first part...
- server with working PHP installation, you'll probably use Apache with mod-php or Nginx with php-fpm. or, with PHP 5.4+, you could use PHP's built-in server (type in your shell: "php -S localhost:8000")
- basic MVC knowledge, but you can do without it

To start writing our blog, we first need to write some routes, views etc. We want to focus on the platform and put the wiring out of the way. We don't want to spend time reinvent features like routing and templating, so we'll just use [Laravel](http://laravel.com) to handle that. In case you haven't heard, Laravel is a lightweight PHP MVC framework. Why Laravel, why not Symfony, Zend , "insert-your-framework-here"? Well, Laravel is simple enough to get you quickly started and has decent documentation. If you don't like it, you are welcome to choose a framework of your choice, be it a full-stack solution like Symfony, or something less complex like Sylex or Slim. Framework is here just for wiring and plumbing, we could write tutorial using bare PHP, but nobody uses that nowadays, right?

## Get started

- Download Laravel at laravel.com
@todo
- Create a new DSL project
- Download the project and initialize as bundle
- Write a simple DSL describing blog post
- Create and display a simple blog post
- Create admin interface

### Short intro to NGS/DSL platform

DSL platform consists of a server running in the cloud, and you don't have to deal with it. Only thing you need is project credentials

### Create a new project

To create your first project, you need to register first. It's a one time Go directly to [registration page](https://dsl-platform.com/register/php) and enter your email
It's time to register at [dsl-platform.com](https://dsl-platform.com/register/php) and create a new project. Change its name to 'Blog' (this will make classes reside in default \Blog namespace). Go to 'Downloads' section and download the zipped project files. Unzip to /bundles/dsl folder in your Laravel installation folder. There should be two folders inside: dsl and platform. Platform holds necessary source files and genertade code. dsl folder will be used to describe our applicaton..

### Register a DSL bundle

We need to make Laravel recognize the DSL bundle, so add following code to /application/bundles.php:

```php
return array(
    'docs' => array('handles' => 'docs'),
    'dsl'  => array('auto' => true),
);
```

The 'auto' option is used to bootstrap bundle. It works by including a start.php file in bundle's root folder. Dsl platform already includes a bootstrap file, so we only need to include it in start.php. Create a start.php file in /bundles/dsl folder and copy inside:

```php
require_once(__DIR__.'/platform/Bootstrap.php');
```

This will bootstrap the entire DSL platform by downloading platform PHP files and generated new models from your DSL files. Notice that registering namespaces for autoloading is not required, as all the necessary files will be included by Boostrap.php.

### Validate your installation

Hello world? Hello...?!
<!---
@todo troubleshooting
---->

## DSL 101

### Writing our first DSL

DSL is a file we'll use to describe our domain model. The first thing our blog needs are posts. Posts should at minimum have have a title and content. Usually, when creating a new application, you would use some sort of ORM to define a Post model that maps to table database. This could look something like:

```php
namespace Blog;

class Post extends MyOrm\Model {
    public $title;
    public $content;
}
```

Then you would go to phpmyadmin and create a table named blog_post. Or you could use your ORM to do that. Maybe you would first create a table in your database and then build a model out of it using tools of your ORM.

In DSL platform, we won't do anything like that. We never need to work directly with the database. Database is running somewhere on our api url, and DSL platform will manage it in the background for us. We need some way to define our models, but we won't use PHP for that. We'll use a DSL for that. All the .dsl files should be placed in /dsl/dsl folder. Let's see how our blog class looks like in DSL. Copy the following lines to /dsl/dsl/Blog.dsl file:

    module Blog
    {
        root Post
        {
            string Title;
            string Content;
            date CreatedAt;
        }
    }

Here we defined a Post object with two string properties, and we also added additional property "CreatedAt" of date type. Keyword 'root' standing before Post is a DSL concept used to define aggregate root objects. Entire DSL platform is founded on Domain driven design. There are other types besides roots, but for now we can put that aside, and just regard a root as a single table in the database. We placed our post inside a Blog module. Module declarations are similar to namespaces.

Ok, that defined Post object in our DSL, what now? Well, now some magic happens. Go to @todo hello-world-link, did something happen? If all you got is hello world, that's okay. <!-- If there's an error, @todo errors --> Check the following folder: /bundles/dsl/platform/modules. In there, you should have a newly created folder Blog (name of our module) and a Post.php file inside it. This is the folder where all our models will be created once they are defined in DSL. Open the Post.php file and take a look at the beginning:

```php
class Post extends \NGS\Patterns\AggregateRoot implements \IteratorAggregate
{
    protected $URI;
    protected $ID;
    protected $Title;
    protected $Content;
    protected $CreatedAt;
}
```

All right, we got our Post class, we can see it extends from AggregateRoot type (remember we defined a root Post in .dsl, which is shorthand for aggregate root). At the top of the class we can see three generated properties defined in DSL. There are also two additional properties: "ID" and "URI". ID is a surrogate key, similar to mysql AUTO_INCREMENT or postgresql SERIAL types. Column ID was automatically generated, because we didn't explicitly state any keys in our Post object. URI property is a special property that holds a unique string representation of primary keys. URI is a unique identifer, in this case, it is equal to ID value since the ID is only key.

So, what exactly happened. On our request to hello-world route, platform was initialized in ngs bundle. On each initialization, platform scans DSL files for changes, that's on each request. If changes are found in .dsl files, platform will use those .dsl files to generate appropriate files. It's similar to some ORMs generating php model classes from tables in database. But you didn't have to define anything in database, entire domain model is contained in DSL.

The generated Blog\Post class can be used to manage posts with basic CRUD operations. We can insert new posts, update and delete them.

### Inserting posts:

Let's try out our new Post class by storing (persisting) a post named 'Hello world'. We'll place code to achieve that in a separate route. To do that, first we'll create a route that creates a 'Hello world' post and persists it. Copy following code at the beginning of routes.php:

```php
Route::get('/create-post', function()
{
    $post = new \Blog\Post();
    $post->Title = 'Hello';
    $post->Content = 'Hello world!';
    $post->persist();

    return 'Created post with URI: '.$post->URI;
});
```

Now fire up your browser and go to {your-localhost-url}/create-post. You should get a response containing "Created post with URI 1001". This means a new Post has been persisted to database. Key ID was assigned with a value of 1001. Numeric keys start at 1000 by default and increase by 1, similarly to auto-incrementing or sequence keys.
Now we need a route to display a particular post. Add this to routes.php:

```php
Route::get('/(:num)', function($uri)
{
    $post = \Blog\Post::find($uri);
    return View::make('home.post')->with('post', $post);
});
```

Let's create a basic view:

/application/views/home.php:

```php
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>DSL blog</title>
</head>
<body>
<?=$content?>
</body>
</html>
```

And a view for post...

/application/views/post.php:

```php
<h1><?=$post->Title?></h1>
<p><?=$post->Content?></p>
```

### Comments

Let's add some comments to our blog. To do thažt, first we must define comments and link them to the post in DSL file. So far, the DSL consists of only one object, Post. Post is an aggregate root, it is an object which encapsulates other objects such as entitites and values. Entity is an object with identity. Entity cannot exist by itself, it must be encapsulated within an aggregate root object. That fits the desription of a comment. Comment by itself is meaningless, each comment is bound to a certain post. To read more about domain driven design, go to [our site](https://docs.dsl-platform.com/ddd-foundations). We can use entities to define objects like comments in following way:

    module Blog
    {
        root Post
        {
            string Title;
            string Content;
            date CreatedAt;
        }

        root Comment
        {
            Post* Post;
            string Email;
            string Content;
            date CreatedAt;

            specification findByPost 'it => it.PostID == PostID'
            {
                int PostID;
            }
    }

Comment[]
Now the Post object contains

something about how a blog without comments

```php
<h1><?=$post->Title?></h1>
<p><?=$post->Content?></p>

<ul>
<? foreach ($comments as $comment): ?>
    <li><?=$comment->Email?> commented: <?=$comment->Content ?></li>
<? endforeach ?>
</ul>

<div>
    <h2>Leave a comment...</h2>
    <form action="/comment" method="post">
        <input type="hidden" name="PostURI" value="<?=$post->URI?>">
        <label>Email: <input type="text" name="Email"></label>
        <br />
        <label>Content: <textarea name="Content"></textarea></label>
        <input type="submit" value="Submit comment">
    </form>
</div>
```

Add route which will persist comment to routes.php

```php
Route::post('/comment', function() {
    $postURI = Input::get('PostURI');
    $comment = new Comment();
    $comment->Post    = Post::find($postURI);
    $comment->Email   = Input::get('Email');
    $comment->Content = Input::get('Content');
    return Redirect::to('/post/'.$postURI);
});
```

something somehtin g persist...

### Filtering results

Lets display comments on each post. So far, we can use findAll method to get all comments and then filter them by a certain postID:

```php
$postID = 1001;
$postComments = array();
foreach(Comment::findAll() as $comment)
    if($comment->PostID == $postID)
        $postComments[] = $comment;
```

This will work, but findAll method fetches all the comments in the database. We need a more efficient way to filter comments before they are fetched. There are two ways to do this: generic search and specificatoions.

### Generic search

By using generic search helper we can construct a specific filter in php:

```php
$postID = 1001;
$sb = new GenericSearch('Blog\Comment');
$comments = $sb->equals('PostID', $postID)->search();
```

GenericSearch constructor receives the name of aggregate root on which the search will be performed.
Calling equals method will instruct search to filter comments with given value for 'PostID' property.
Search method invokes a post request that returns results.


### Specifications

More DDD style would be to use specification to filter out results for each post.
Specification states a certain condition, here's a specificiation we'll use to find all comments belonging to a certain post. blog.dsl:

    root Comment
    {
        Post* Post;
        string Email;
        string Content;
        date CreatedAt;

        specification findByPost 'item => item.PostID == PostID'
        {
            int PostID;
        }
    }

Each specification has a name ("findByPost"). Inside single quotes there is a lambda expression, curly braces hold an argument PostID passed to speicification. Lambda is used to state  a condition. Expression in condition is evaluated  for each Comment. We can now use this specification to get only commmetns for which the specifiaction is true. To do that, platform generates a static function findByPost inside Blog\Post class. Calling it will fetch comments for specific post:

```php
$postID = 1001;
$results = Comment::findByPost($postID);
```

If you aren't familiar with lambdas, you can think of it in PHP terms as passing anonymous function to [array_filter](http://php.net/array_filter), somewhat like this: (just an example, don't do this)

```php
<?php
$postID = 1001;
$results = array_filter(Comment::findAll(), function($item) use ($postID) {
    return $item->PostID === $postID;
})
```

Array_filter will return each element for which the passed in closure (lambda in specification) returns true.
Syntax "use ($postID)" ensures that variable $postID is imported into function scope. Otherwise, variable could not be used inside function. Again, this serves only as example to explain specification behaviour.
@todo

$results will hold only those values for which callback passed to array_filter returns true.
Of course, the difference is that comments are filtered on the server with specification.
@todo

To call this specification from php:


Specification will return an array of Blog\Comment objects.
You can look for implementation details in generated code (Blog/Comment.php in /dsl/platform/modules folder);

```php
Route::get('/post/(:num)', function($postURI)
{
    $post = \Blog\Post::find($postURI);
    $comments = \Blog\Comment::findByPost($postURI);
    $view = View::make('base');
    return $view->nest('content', 'post', array(
        'post'     => $post,
        'comments' => $comments,
    ));
});
```

Specifications can be used without arguments. For example, 

    root Comment
    {
        Post* Post;
        string Email;
        string Content;
        date CreatedAt;

        specification findByPost 'item => item.PostID == PostID'
        {
            int PostID;
        }
    }

### Deleting comments

A small example route to show how to delete a comment:

    ```php
    Route::get('/comment/delete/(:id)', function($id)
    {
        $comment = \Blog\Comment::find($uri);
        $comment->delete();
        return Redirect::back(); // referer
    });
    ```

You can use this for deleting any kind of root.
If a certain comment doesn't exist, find method will throw an exception. You should add handling to each route, or global handler...

    ```php
    Route::get('/comment/delete/(:id)', function($id)
    {
        try {
            $comment = \Blog\Comment::find($uri);
            $comment->delete();
            return Redirect::back(); // referer
        }
        catch (\NGS\Client\NotFoundException\Exception $ex) {
            return Response::make($ex->getMessage(), 404);
        }
    });
    ```

Now we'll add a small admin section for managing posts. 

### Login, logout...

Usually we need some sort of login to access administaration. DSL platform is not intended t o provide authentication. That should be up to your framework, CMS, single-file app or whatewer you are using.
We'll provide a simple snippet that gives you an idea of how you can store user data, which is basis for further authentitcation. Concrete implemenation is left out to specific platform or framework ant that is out of scope of this tutorial. 
Putting "Username" property in parenthesis after root name will make Username a  primary key. Users can now be referenced by that username as its URI, and not some autogenerated surrogate key.

A most basic dsl, with only one root for handling our users.

    module Security
    {
        root User (Username)
        {
            string Username;
            string Password;
        }
    }

A login route where we check if posted username and password match any user.
    
    ```php
    Route::post('/login', function() {
        $username = Input::post('username');
        $password = Input::post('password');
        try {
            $user = User::find($username);
            // in real life, you would use some sort of password hashing
            if ($user->Password === $password) {
                // credentials are ok
                // do authentication stuff, store login in session, etc.
            }
            else {
                // login has failed
            }
        }
        catch (\NGS\Client\Exception\NotFoundException $ex) {
            // user with specified username does not exist
        }
        $user = Security\User()::find()
    });
    ```

### DSL and beyond

There is much more to DSL platform than what can be showcased in a simple blog application.
Here we'll show some more advanced feature: OLAP analysis.

[OLAP](http://en.wikipedia.org/wiki/Online_analytical_processing) can be used to analyse data.


