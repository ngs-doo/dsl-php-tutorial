# DSL platform Tutorial

## Introduction

DSL platform is a multi-language platform inspired by Domain driven design.

<!-- @todo uljepsati uvod  -->

In this tutorial we'll show how you can rapidly develop a small PHP blog application by using DSL platform.

## Setup

### What you'll need:

- PHP 5.3.7 or greater
- basic OOP, you should know how to write your own class, create a new object instance and call its methods, that should do it for the first part...
- server with working PHP installation, you'll probably use Apache with mod-php or Nginx with php-fpm. or, with PHP 5.4+, you could use PHP's built-in server (type in your shell: "php -S localhost:8000")
- basic MVC knowledge, but you can do without it
- Git

To start writing our blog, we first need to write some routes, views etc. We want to focus on the platform and put the wiring out of the way. We don't want to spend time reinvent features like routing and templating, so we'll just use [Laravel](http://laravel.com) to handle that. In case you haven't heard, Laravel is a lightweight PHP MVC framework. Why Laravel, why not Symfony, Zend , "insert-your-framework-here"? Well, Laravel is simple enough to get you quickly started and has decent documentation. If you don't like it, you are welcome to choose a framework of your choice, be it a full-stack solution like Symfony, or something less complex like Sylex or Slim. Framework is here just for wiring and plumbing, we could write tutorial using bare PHP, but nobody uses that nowadays, right?

<!--
## Get started

- Git
- [Composer](https://getcomposer.org/download)
- Create a new DSL project 
- Download the project and initialize as bundle
- Write a simple DSL describing blog post
- Create and display a simple blog post
- Create admin interface

### Short intro to NGS/DSL platform

DSL platform consists of a server running in the cloud, and you don't have to deal with it. Only thing you need is project credentials

-->

### Installation

1. Install basic Laravel application
    You can fork the basic skeleton application from [Github](https://github.com/nutrija/dsl-php-tutorial) and then clone it, or go to your projects folder and clone the original repository directly:

        $ git clone git@github.com:nutrija/dsl-php-tutorial dslblog

    This will clone your repository into a new 'dslblog' folder. Repository contains this readme file, composer.json and basic Laravel file structure. We'll use composer to install Laravel core files. If you have composer installed, you can run 'composer install' inside dslblog folder. If you haven't used composer yet, composer is a PHP package manager that will download and register your dependencies. It's simple to install it, go to dslblog and run:
        
        $ cd dslblog
        $ curl -sS https://getcomposer.org/installer | php
        $ php composer.phar install

    This will download and execute composer.phar file which will download all the required Laravel dependencies.
    
    Next, you should make app/storage folder writable by the server user. If you are running apache as www-data user, you can run:

        $ chown www-data. app/storage/ -R

2. Setup server

    Now you should setup your server to point to public folder. If you are using Apache, you can place a symlink to dslblog/public folder in your server root or setup in some other way.
    Open your browser and go to http://localhost/dslblog. You should see 'Hello world' text. That means Laravel installation is working. Next, we'll setup DSL platform.

3. Install DSL platform

    To install DSL platform, you'll first need to register and create a project at [dsl-platform.com] (https://dsl-platform.com). Each application should have a separate project, similar to application having a separate database. To create your first project, you need to register. It's a simple one-step process, just go to [registration page](https://dsl-platform.com/register/php) and enter your email. You'll be automatically logged in.

    After registration, you should be logged in. Click on the 'New project' button. This will create an empty new project. Now you can change its name to 'Blog' (this will make model classes reside in default \Blog namespace). Go to 'Downloads' section and download the zipped project files. Create a folder named 'dsl-platform' in '/app' folder of your Laravel installation and extract the download there. Now you should have two folders inside: dsl and platform. Platform folder contains necessary source files and generated code. The 'dsl' folder will be used to store .dsl files in which we'll describe our application data domain (models).

    We need to make Laravel recognize the DSL bundle. Normally, we'd make a separate bundle which handles initialization, but we'll skip that here and just glue registration at the beginning of start/global.php file:

    ```php
        require_once(__DIR__.'/../dsl-platform/platform/Bootstrap.php');
    ```

    Including bootstrap is all you need to get the platform running. Bootstrap will handle:
        
        - downloading and updating platform core source files
        - downloading generated model files (these will be generated from description inputted in DSL files, we'll soon get to details)
        - class loading (no autoloading is required)
        - initializing connection to the server

    Bootstrap needs write permissions to platform folder to copy downloaded source files. To set up permissions with apache as www-data user, run in your terminal:

    ```bash
        $ chown www-data. app/dsl-platform/platform/ -R
    ```

### Validate your installation

    Open browser, and enter url or path to your installation. You should get a "Hello, world!" response. That means everything is up and running.

## DSL 101

We're creating a blog application, so let's get started with creating some posts. 
Usually, if starting from scratch, we would first create a database and a table named 'Blog' and then use some sort of ORM to define a Post class with fields mapped to 'Blog' table columns. Each post needs a title and a content, so we'll end with a Blog class that looks something like this:

```php
namespace Blog;

class Post extends MyOrm\Model {
    public $title;
    public $content;
}
```

We could also use annotations or properties to specify constraints, validations, relationships with other entities etc. Depending on the tools we use, we could specify those settings in database and then build our model classes, or vice versa, write PHP and let ORM update the database. In worst case, we'll have to update both models and database ourselves.

In DSL platform, we'll do things a little different. First, there is no need for us to know anything about the database and we'll never work directly with it. Database will be running somewhere on the cloud, and DSL platform will manage it in the background for us. So, without the database, we need some way to define our data models. We could do that in PHP, but, well, PHP really wasn't intended to describe domain models and relationships, so we'll use a more suitable tool for that, we'll just call it DSL. DSL stands for [Domain specific language](http://en.wikipedia.org/wiki/Domain-specific_language). In case you haven't heard or worked with some kind of DSL, think again, if you ever wrote a single line of database query in SQL, you've used DSL, as SQL is also one kind of DSL.

### Writing our first DSL

Let's get started by translating our post to DSL. Some conventions: all the .dsl files should be placed in /dsl-platform/dsl folder. Let's see how our blog class looks like in DSL. Copy the following lines to /dsl/dsl/Blog.dsl file:

    module Blog
    {
        root Post
        {
            string title;
            string content;
        }
    }

This DSL is pretty straightforward: it tells the platform to define a Post object with two string properties. Keyword 'root' standing before Post is a DSL concept used to define an aggregate root. Aggregate root is one of core concepts used in [Domain driven design](http://en.wikipedia.org/wiki/Domain-driven_design) (DDD). DSL platform is founded on DDD concepts. There are other types besides roots, but for now we can put that aside, we'll just be using 'root' as a basic building block. For now, you can just regard a root as a single table in the database.

First line is used to define a module. Modules are used to encapsulate objects and avoid name collisions, pretty similar to PHP namespaces. So, the first line tells the platform that Post object will be placed inside a Blog module.

That defined a Post object in our DSL, what now? Open your browser and go to the URL of your application (the same you pointed to Laravel public folder to verify your installation). As before, you should get a 'Hello world' response. Currently we cannot see any difference here because the default controller returns the same response, we'll change that soon. But first, let's check what happened in the background. Check the contents of platform/modules in the dsl-platform folder. There should be a newly created Blog folder (taken from module name) and a Post.php file inside it. Those files were created using the description we provided in blog.dsl.

Each time any .dsl file is modified, platform will send the DSL contents to the platform server. Server will use definitions in DSL to update the database and generate PHP files representing concepts defined in DSL. The platform/modules is a folder that will contain all the generated PHP classes. It's similar to some tools generating PHP by using only database. In our case, we're using DSL to generate both PHP files and the database. To make things clearer, take a look at the beginning of generated Post.php file:

```php
class Post extends \NGS\Patterns\AggregateRoot implements \IteratorAggregate
{
    protected $URI;
    protected $ID;
    protected $title;
    protected $content;
}
```

All right, we got our Blog\Post class, we can see it extends from AggregateRoot class (remember we defined a Post as 'aggregate root' concept in blog.dsl). At the top of the class there are two properties that were defined in DSL: title and content. There are also two additional properties: "ID" and "URI". ID is a surrogate key, similar to AUTO_INCREMENT or SERIAL types found in mysql and postgresql. Column ID was automatically generated, because there were no explicitly defined keys inside of root Post (you could do that, we'll get to that later). Each time a new Post is inserted, ID will get the value of last inserted ID increased by 1.

Another property we didn't specify is URI. URI is a special property that serves as a unique identifier. It holds a unique string representation of primary keys. With only one key (ID), URI value will be the same to ID (if we ignore types, since ID is an integer, and URI is a string).

The generated Blog\Post class can now be used to handle CRUD operations like inserting new posts or updating and deleting existing ones:

### Inserting posts:

Let's write some PHP code. We can try out our new Post class by storing (persisting) a post named 'Hello world' to the database. To do that, first we'll create a route that creates a 'Hello world' post and persists it. Copy the following code on the beginning of routes.php file: (for convenience and simplicity, we'll just stuff all the PHP in that file for now, which is not a best idea for a real-world application)

```php
Route::get('/test', function()
{
    $post = new \Blog\Post();
    $post->title = 'Hello';
    $post->content = 'Hello world!';
    $post->persist();

    return 'Created post with URI: '.$post->URI;
});
```

Now fire up your browser and go to {your-url}/test. You should get a response containing "Created post with URI 1001". This means a new Post has been persisted to database. It was assigned ID with value equal to 1001. That's because numeric keys start at 1000 by default and increase by 1, similarly to auto-incrementing or sequence keys.

The code is self-descriptive: there is a new instance created, and some values are assigned to both Title and Content properties. After that, the persist method will start the communication with the server and tell it to store the object. Persist method is used both for inserting and updating. Since we created a new post instance which doesn't exist in database yet, a new post will be inserted into database. 

Now we need to display an existing post. For that we'll create a route that searches for a specific post and displays it in a view (template) file. Add that route to routes.php:

```php
Route::get('/(:num)', function($uri)
{
    $post = \Blog\Post::find($uri);
    return View::make('home.post')->with('post', $post);
});
```

The ::find($uri) method will query the database for a Blog\Post with specified URI value. This method can be used on every aggregate root. If the object is not found, an exception will be thrown.
Let's create a (very) basic view for site's html layout:

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

And another view for post...

/application/views/post.php:

```php
<h1><?=$post->title?></h1>
<p><?=$post->content?></p>
```

Now you should see the post contents at {your-url}/1001.

### Specifying relationships

Each blog should have some comments functionality, so let's add that. To do that, first we must describe a comment in the DSL. Let's expand our DSL with a Comment:

    module Blog
    {
        root Post
        {
            string title;
            string content;
        }

        root Comment
        {
            string email;
            string content;
            date createdAt;
        }
    }

We added Comment as another root (that's the only DDD type we'll use for now). A basic comment has an email and content, and it should be useful to know when the comment was entered, so we specified a property named createdAt of 'date' type. Besides simple (or primitive) types such as strings, you can specify properties as complex types. Date is just one of the complex types you can use in the platform. In generated PHP code, date will be an instance of NGS\LocalDate class.

If we refresh our blog, we'll see that Comment.php and some more files were generated and copied into modules folder. That means we can persist and read comments as we did with posts. That won't make much sense if we can't somehow connect each comment to a specific post. One way to do that is to specify a reference in each comment which specifies a particular post to which the comment belongs to. We can do that by adding a single line inside Comment:
    
    root Comment
    {
        Post* post;
        string email;
        string content;
        date createdAt;
    }

This will create a new property called Post inside the Comment class. It serves as a reference to an existing Post instance and we can use it to link comments and posts.
Now let's expand the post view with comments and a form to post new comments:

/application/views/post.php:

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

And here's a route which will persist the comment posted from the form:

```php
Route::post('/comment', function() {
    $postURI = Input::get('PostURI');
    $comment = new Comment();
    $post = Post::find($postURI);
    $comment->post    = $post;
    $comment->email   = Input::get('Email');
    $comment->content = Input::get('Content');
    $comment->persist();
    return Redirect::to('/post/'.$postURI);
});
```

The function find in Post::find($postURI) will search for a post with specified URI and return an instance of Post object. That object can then be assigned as reference in $comment->post. That reference will be persisted to database.

## Fetching objects

Each root class has a findAll method which can be used to load all stored items. Let's use it to display comments on each post:

```php
$postID = 1001;
$postComments = array();
foreach(Comment::findAll() as $comment)
    if($comment->PostID == $postID)
        $postComments[] = $comment;
```

That will work, but that's not very efficient because findAll method fetches all the comments in the database. We need a more efficient way to filter comments before they are fetched. There are two ways to do this: generic search and specifications.

### Generic search

By using generic search helper we can construct a filter which will return only comments pointing to a certain post:

```php
$postID = 1001;
$search = new GenericSearch('Blog\Comment');
$comments = $search->equals('PostID', $postID)->search();
```

GenericSearch constructor receives the name of aggregate root on which the search will be performed.
Equals method is one of the possible filters. It specifies exact match where the first argument is property name and the second argument is matched value.

### Specifications

Specification states a certain condition, it has various uses, one of them is to filter items. We write a specification in the DSL. Here's a specification that fetches all comments belonging to a certain post:

    root Comment
    {
        Post* post;
        string email;
        string content;
        date createdAt;

        specification findByPost 'item => item.PostID == postID'
        {
            int postID;
        }
    }

This creates a specification called findByPost. Specification consists of expression and the optional list of arguments. Expression specified behind the name is called a lambda expression.
Lambda expression is used to state a condition which evaluates as true or false for each Comment. The condition we specified defines that condition is true for all comments that have PostID property equal to the specified postID argument. The line 'int postID' specifies the argument which the specification receives. This is the argument which we pass to specification in PHP:

```php
$postID = 1001;
$results = Comment::findByPost($postID);
```
This will fetch all the comments for post with ID 1001. $results will hold an array of Blog\Comment objects.
If you aren't familiar with lambdas, you can think of it in PHP terms as passing anonymous function to [array_filter](http://php.net/array_filter), like this:

```php
$postID = 1001;
$results = array_filter(Comment::findAll(), function($item) use ($postID) {
    return $item->PostID === $postID;
})
```

Array_filter will iterate through each comment that is returned from Comment::findAll() (every comment in the database!) and return each element for which the passed in closure (similar to lambda in specification) returns true.
You'll get the same results, but this serves just an example - you should never run code like this in production, as it is very inefficient to fetch all the comments from the server and only then filter them, and that's the reason why we write a specification inside DSL.

We can now expand a route that displays post with specification for finding comments.

```php
Route::get('/(:num)', function($postURI)
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

### Deleting comments

Here's an example route showing how to delete a comment:

```php
Route::delete('/comment/(:uri)', function($uri)
{
    $comment = \Blog\Comment::find($uri);
    $comment->delete();
    return Redirect::back(); // referer
});
```

Similar code can be used to delete any kind of root. If a comment wasn't found, the find method will throw an exception which should be handled in a try-catch block inside the route, or alternatively, you could handle those types of exceptions somewhere outside in global handler. Here's how to do it in the route:

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

### Login, logout...

DSL platform is not intended to provide authentication. This is a job for a PHP framework or a standalone component. DSL platform can be integrated with those components and store authentication data.
We'll provide a simple snippet that gives an idea of how to store user data, which can be basis for further authentication.

This is a simple DSL for handling user data:

    module Security
    {
        root User (Username)
        {
            string Username;
            string Password;
        }
    }

(obviously, you should not store passwords as plaintext)

Putting "Username" property in parenthesis after root name defines the property as a primary key. Users can now be uniquely identified by that username (it will equal URI property), and there is no more need for a ID property.

A simple login route where we check if posted username and password match any user.
    
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

## DSL and beyond

There are many more features of DSL platform, such as generating reports in various formats and analyzing data using built-in [OLAP](http://en.wikipedia.org/wiki/Online_analytical_processing) features.

Besides the features built-in in the platform, the DSL platform brings certain features specifically to PHP. One of the more interesting is type safety, which is a way to improve PHP's type system.

### PHP type safety

PHP classes generated from the DSL will provide some type safety. Let's assume the following DSL:

module Blog
{
    root Post {
        string title;
    }
}

What happens when an illegal operation is performed, such as assigning an array to title property:

```php
$post = new Blog\Post();
$post->title = array();
```

This snippet will throw an InvalidArgumentException explaining that an invalid type is being assigned to a string property. Under the hood, each property is set using PHP's magic setter function __set which calls an appropriate setter function, in this case the setTitle function. The setter checks type of value being assigned to the property and raises an exception if that type cannot be converted to the type specified in DSL.
