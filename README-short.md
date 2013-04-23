# DSL platform Tutorial

## Introduction

This tutorial will show you how to develop a small PHP blog application by using DSL platform.

## Setup

1. Clone the repository

    You can clone or fork&clone the basic application from [Github](https://github.com/nutrija/dsl-php-tutorial).

        $ git clone git@github.com:nutrija/dsl-php-tutorial dslblog

2. Install dependencies via composer

    Run composer to install dependencies:
        
        $ cd dslblog
        $ curl -sS https://getcomposer.org/installer | php
        $ php composer.phar install

    Setup permissions:

        $ chown www-data. app/storage/ -R

3. Server setup

    Setup your server so it's document root points to the dslblog/public folder.
    Open your browser and go to http://localhost/dslblog. You should see a 'Hello world' message.

4. Install DSL platform

    Register at [dsl-platform.com] (https://dsl-platform.com). Then create a project and download the zip from the 'Downloads' section. Extract the download to the '/app/dsl-platform' folder. 

    To load the platform, add at the beginning of start/global.php file:

    ```php
        require_once(__DIR__.'/../dsl-platform/platform/Bootstrap.php');
    ```

    Platform will need write permissions for the /app/dsl-platform/platform/ folder to download generated source files. For server running as www-data, set permissions like this:

    ```bash
        $ chown www-data. app/dsl-platform/platform/ -R
    ```

5. Check installation

    Open browser, and enter url or path to your installation. You should get a "Hello, world!" response. That means everything is up and running.

## DSL 101

We'll describe our models in DSL files. DSL files must be placed in /dsl-platform/dsl folder. Let's describe a blog post in DSL:

    module Blog
    {
        root Post
        {
            string title;
            string content;
        }
    }

This DSL defines an aggregate root object named 'Post' that contains two string properties. Aggregate root is one of core concepts used in [Domain driven design](http://en.wikipedia.org/wiki/Domain-driven_design) (DDD). We'll be using this concept as a basic building block. For now, you can just regard a root as a single table in the database.

Each time a DSL file is modified, platform will use it to update the database and generate PHP source files which can be found in the platform/modules folder. Go there and take a look at the beginning of generated Post.php file:

```php
class Post extends \NGS\Patterns\AggregateRoot implements \IteratorAggregate
{
    protected $URI;
    protected $ID;
    protected $title;
    protected $content;
}
```

Title and content properties were specified in the DSL. URI and ID are generated properties, they are used as object identifiers.
We can use this class to perform various operations, e.g. create and persist a new Post object. 

### Persisting data:

We'll create a route that persists a 'Hello world' post. Copy the following code on the beginning of routes.php file:

```php
Route::get('/test', function()
{
    $post = new \Blog\Post();
    $post->title = 'Hello';
    $post->content = 'Hello world!';
    $post->persist();

    return 'Created a new post. Post URI is: '.$post->URI;
});
```

(*note: it's not best practice to use HTTP GET methods to persist data, this is just an example)

Now go to http://localhost/test (or your server URL). You should get a response containing "Created post with URI 1001". That means a new Post has been persisted to database. It was assigned ID with value equal to 1001. (numeric keys start increment starting from 1000) 

Inside the 'persist' method, a HTTP request is sent to the server, which persist the data to database, and then returns a generated URI value.

### Reading persisted data:

Let's create a route that searches for a post with specific URI and passes it to view:

```php
Route::get('/(:num)', function($uri)
{
    $post = \Blog\Post::find($uri);
    return View::make('home.post')->with('post', $post);
});
```

The 'find' method queries the database for a Blog\Post with specified URI value. If the object is not found, an exception will be thrown.
Let's create a view for basic site layout:

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

Now you should see the post contents at http://localhost/1001.

### Specifying relationships

One way to specify a relationship is to specify a property as a reference to another object. We'll add a Comment that points to a post:
    
    root Comment
    {
        Post* post;
        string email;
        string content;
        date createdAt;
    }

Property named 'post' points to a specific Post instance. Now let's expand the post view with comments and a form to post new comments:

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

### Finding objects

Each root class has a findAll method which retrieves all persisted objects:

```php
$comments = Comment::findAll();
```

### Generic search

Retrieve only those comments pointing to a certain post:

```php
$postID = 1001;
$search = new GenericSearch('Blog\Comment');
$comments = $search->equals('PostID', $postID)->search();
```

GenericSearch equals method is just one of many filter methods that can be used to customize search.

### Specifications

Specification states a certain condition. It has various uses, one of them is to filter retrieved items. Here's a specification that fetches all comments belonging to a certain post:

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

Use this specification in PHP to find all comments for a certain post:

```php
$postID = 1001;
$results = \Blog\Comment::findByPost($postID);
```

We can now expand a route that displays post with this specification:

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

### Deleting data

Here's a route that deletes a certain comment:

```php
Route::delete('/comment/(:uri)', function($uri)
{
    $comment = \Blog\Comment::find($uri);
    $comment->delete();
    return Redirect::back(); // referer
});
```

The delete method can be used on every root class. If the specified object is not found, exception will be thrown which you can handle like so:

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

Title is a string property, and only strings should be assigned to it. If an illegal operation is performed by assigning it an invalid type such as array property:

```php
$post = new Blog\Post();
$post->title = array();
```

... an InvalidArgumentException is thrown. Under the hood, each property is assigned in a setter method called using magic __set function. Each setter will raise an exception if value being assigned to the property cannot be converted to the analogous type specified in DSL.
