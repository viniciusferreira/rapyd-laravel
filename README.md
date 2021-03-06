rapyd-laravel
=============

This is a pool of presentation and editing widgets (Grids and Forms) for laravel 4.  
Nothing to "generate", just some classes to let you develop and maintain CRUD backends in few lines of code.
Main Website: [rapyd.com](http://www.rapyd.com)
Demo: [rapyd.com/demo](http://www.rapyd.com/demo)

![rapyd laravel](https://raw.github.com/zofe/rapyd-laravel/master/public/assets/rapyd-laravel.png)

## DataSet

DataSet can paginate results starting from query, eloquent collection or multidimensional array.  
It add the ability to order result and keep persistence of all params in query string.

i.e.:
```
/dataset/uri?page=2&ord=-name   will get page 2 order by "name" descending
/dataset/uri?page=3&ord=name&other=xx  will get page 3 order by "name" and keeping "other=xx"
```

in a controller 

```php
   //using table name
   $dataset = DataSet::source("tablename")->paginate(10)->getSet();

   //or using query
   $dataset = DataSet::source(DB::table('users')->select('name', 'email'))->paginate(10)->getSet();

   //or using eloquent model or eloquent builder 
   $dataset = DataSet::source(new Article)->paginate(10)->getSet();
   $dataset = DataSet::source(Article::with('author'))->paginate(10)->getSet();

   //or using array
   $dataset = DataSet::source($multidimensional_array)->paginate(10)->getSet();
```

in a view you can use

```php
<p>
    //cycle
    @foreach ($dataset->data as $item)

        //field
        {{ $item->title }}<br />
        //firld from relation
        {{ $item->author->name }}<br />

    @endforeach

    //pagination links
    {{ $dataset->links() }} <br />

    //sort link
    {{ $dataset->orderbyLink('title', 'asc') }} <br />
</p>
```



## DataGrid

DataGrid extend DataSet to make data-grid output with few lines of fluent code.  
It build a bootstrap striped table, with pagination at bottom and order-by links on table header.
It support also  blade syntax inline. 

in a controller 

```php
   $grid = DataGrid::source(Article::with('author'));  //same source types of DataSet
   $grid->add('title','Title', true); //field name, label, sortable
   $grid->add('{{ substr($body,0,20) }}...','Body'); //blade syntax
   $grid->add('{{ $row->author->name }}','Author'); //blade syntax with related field
   $grid->edit('/dataedit/uri', 'Edit','modify|delete'); //shortcut to link DataEdit actions
   $grid->paginate(10);

   View::make('articles', array('grid'=>$grid->getGrid()))

   //since we use also __toString() you can do..
   ...
   View::make('articles', compact('grid'))

```

in a view you can just write

```php
  #articles.blade.php
  {{ $grid }}
```

## DataForm

 DataForm is a form builder, you can add fields, rules and buttons.  
 It will build a bootstrap form, on submit it  will check rules and if validation pass it'll store new entity.  

```php
   //start with empty form to create new Article
   $form = DataForm::source(new Article);
   
   //or find a record to update some value
   $form = DataForm::source(Article::find(1));

   //add fields to the form
   $form->add('title','Title', 'text'); //field name, label, type
   $form->add('body','Body', 'textarea')->rule('required'); //validation

   //you can also use shorthand methods, add{Type}(...
   $form->addText('title','Title'); //field name, label
   $form->addTextarea('body','Body')->rule('required');

   //then a submit button
   $form->submit('Save');

   //at the end you can use closure to add stuffs or redirect after save
   $form->saved(function() use ($form)
   {
        $form->message("ok record saved");
        $form->link("/another/url","Next Step");
   });

   View::make('article', compact('form'))
```

```php
   #article.blade.php
  {{ $form }}
```

note: DataForm can also work without entity, just as Form builder, use __DataForm::create()__ instead of DataForm::source in this case

## DataEdit
  DataEdit extends DataForm, it's a full CRUD application for given Entity.  
  It has status (create, modify, show) and actions (insert, update, delete) 
  It detect status by simple query string semantic:


```
  /dataedit/uri                     empty form    to CREATE new records
  /dataedit/uri?show={record_id}    filled output to READ record (without form)
  /dataedit/uri?modify={record_id}  filled form   to UPDATE a record
  /dataedit/uri?delete={record_id}  perform   record DELETE
  ...
```

```php
   //simple crud for Article entity
   $edit = DataEdit::source(new Article);
   $edit->add('title','Title', 'text')->rule('required');
   $edit->add('body','Body','textarea')->rule('required');
   $edit->add('photo','Photo', 'file')->rule('image')->move('uploads/');
   return $edit->view('crud', compact('edit'));

```

```php
   #crud.blade.php
  {{ $edit }}
```

note: we use _$edit->view_  method  instead _View::make_ for a reason: DataEdit must manage  redirects. With other widgets you should use View facade as default.    

## DataFilter
DataFilter extends DataForm, each field you add and each value you fill in that form is used to build a __where clause__ (by default using 'like' operator).   
It should be used in conjunction with a DataSet or DataGrid to filter results.  


```php
   $filter = DataFilter::source(new Article);
   $filter->add('title','Title', 'text');
   $filter->submit('search');
   $filter->reset('reset');
   
   $grid = DataGrid::source($filter);
   $grid->add('nome','Title', true);
   $grid->add('{{ substr($body,0,20) }}...','Body');
   $grid->paginate(10);

   View::make('articles', compact('filter', 'grid'))
```
```php
   # articles.blade
   {{ $filter }}
   {{ $grid }}
```
## Install 


To `composer.json` add: `"zofe/rapyd": "1.0.*"`
and then run: `$ composer update zofe/rapyd`.

In `app/config/app.php` add this service provider: `'Zofe\Rapyd\RapydServiceProvider',`.


## Publish & integrate assets

`php artisan asset:publish zofe/rapyd`

then you need to add this to your views, to let rapyd add runtime assets:

```php
<head>
  ...
    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.3/css/bootstrap.min.css">
    <script src="//netdna.bootstrapcdn.com/bootstrap/3.0.3/js/bootstrap.min.js"></script>
    <script src="http://code.jquery.com/jquery-1.10.1.min.js"></script>
   {{ Rapyd::head() }}
</head>
```
note: widget output is in standard with __Boostrap 3+__, and some widget need support of __JQuery 1.9+__
so be sure to include dependencies as above



## In short

Rapyd use a "widget" approach to make a crud, without "generation".
(this approach is worst in terms of flexibility but fast/rapid in terms of development and maintenance):

_You need to "show" and "edit" record from an entity?_  
Ok so you need a DataGrid and DataEdit.
You can build compoments where you want (even multiple widgets on same route).
An easy way to work with rapyd is:
  * make a route to a controller for each entity you need to manage
  * make the entity controller with one method for each widget (one for a datagrid and one for a dataedit)
  * make an empty view, include bootstrap and display content that rapyd will build for you


Would you like to see a sample Backend ?  bum:

/app/routes.php
```php
...
Route::controller('admin', 'AdminController');
```

/app/controllers/AdminController.php
```php
class AdminController extends BaseController {

	public function getArticles()
	{
        $content = DataGrid::source( Article::with("user"));
        $content->link('/admin/article', "New Article",  "TR");
        $content->add('title','Title', true);
        $content->add('{{ substr($body,0,20) }}...','Body');
        $content->add('{{ $row->user->email }}','author');
        $content->edit('/admin/article', 'Edit','modify|delete');
        $content->paginate(10);
        return  View::make('admin.edit', compact('content'));
	} 

	public function anyArticle()
	{
        $content = DataEdit::source(new Article);
        $content->link('/admin/articles', "Article List",  "TR");
        $content->add('title','Title', 'text')->rule('required');
        $content->add('body','Body', 'redactor');
        $content->add('user_id','Author','select')->options(User::lists("username", "id"));
        return $edit->view('admin.edit', compact('content'));
	} 
 
}
```

/app/views/admin/edit.blade.php
```php
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <link  href="//netdna.bootstrapcdn.com/bootstrap/3.0.3/css/bootstrap.min.css" rel="stylesheet">
    <script src="//netdna.bootstrapcdn.com/bootstrap/3.0.3/js/bootstrap.min.js"></script>
    <script src="http://code.jquery.com/jquery-1.10.1.min.js"></script>
    {{ Rapyd::head() }}
  </head>
  <body>
    <div id="wrap">
      <div class="container">
        {{ $content }}
      </div>
    </div>
  </body>
</html>
```



## License

Rapyd is licensed under the [MIT license](http://opensource.org/licenses/MIT)

If Rapyd saves you time, please consider [tipping via gittip](https://www.gittip.com/zofe)