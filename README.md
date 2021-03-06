![Eloquent Table Banner]
(https://raw.githubusercontent.com/stevebauman/eloquent-table/master/eloquent-table-banner.jpg)

[![Travis CI](https://travis-ci.org/stevebauman/eloquent-table.svg?branch=master)](https://travis-ci.org/stevebauman/eloquent-table)
[![Code Climate](https://codeclimate.com/github/stevebauman/eloquent-table/badges/gpa.svg)](https://codeclimate.com/github/stevebauman/eloquent-table)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/stevebauman/eloquent-table/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/stevebauman/eloquent-table/?branch=master)
[![Latest Stable Version](https://poser.pugx.org/stevebauman/eloquenttable/v/stable.svg)](https://packagist.org/packages/stevebauman/eloquenttable) 
[![Total Downloads](https://poser.pugx.org/stevebauman/eloquenttable/downloads.svg)](https://packagist.org/packages/stevebauman/eloquenttable) 
[![Latest Unstable Version](https://poser.pugx.org/stevebauman/eloquenttable/v/unstable.svg)](https://packagist.org/packages/stevebauman/eloquenttable) 
[![License](https://poser.pugx.org/stevebauman/eloquenttable/license.svg)](https://packagist.org/packages/stevebauman/eloquenttable)

An HTML table generator for laravel collections

##Eloquent-Table

###Installation

Include the package in `composer.json`:

    "stevebauman/eloquenttable": "1.0.*"

Now perform a `composer update`.

Include the service provider in the <em>bottom</em> `app.php` config file:

    'Stevebauman\EloquentTable\PaginationServiceProvider',
    'Stevebauman\EloquentTable\EloquentTableServiceProvider',

Insert the trait on your model:
    
    class Book extends Eloquent {

        use \Stevebauman\EloquentTable\TableTrait;

        protected $table = 'books';

    }

Publish the config file (optional)

    php artisan config:publish stevebauman/eloquenttable

You're good to go!

###Usage
    
Grab records from your model like usual:

    $books = Books::get();

    return view('books.index', compact('books'));

Inside your blade view, we just specify the columns we want to show, and then call the render method:

    {{ $books->columns(array(
                'id' => 'ID',
                'title' => 'Title',
                'author' => 'Authored By'
            ))
            ->render() 
    }}

#####Handling relationship values using `means($column, $relationship)`:

    {{ $books->columns(array(
                'id' => 'ID',
                'title' => 'Title',
                'author' => 'Authored By',
                'owned_by' => 'Owned By',
            ))
            ->means('owned_by', 'user.first_name')
            ->render()
    }}

The model books, needs to have a user method defining it's relation for this to work.

You must also use 'dot' notation to indicate the relationship.

#####Customizing the display of the column value using `modify($column, $closure)`:

    {{ $books->columns(array(
                'id' => 'ID',
                'title' => 'Title',
                'author' => 'Authored By',
                'owned_by' => 'Owned By',
            ))
            ->means('owned_by', 'user')
            ->modify('owned_by', function($user, $book) {
                return $user->first_name . ' ' . $user->last_name;
            })
            ->render() 
    }}

Using modify, we can specify the column we want to modify, and the function will return the current relationship record (if the column is a relationship),
as well as the current base record, in this case the book.

#####With eloquent-table, we can also generate sortable links for columns easily:

In your controller:

    $books = Book::sort(Input::get('field'), Input::get('sort'))->get();


In your view:

    {{ $books->columns(array(
                'id' => 'ID',
                'title' => 'Title',
                'author' => 'Authored By',
                'owned_by' => 'Owned By',
            ))
            ->sortable(array('id', 'title'))
            ->render()
    }}

A link will be generated inside the column header that will be clickable. The HTML generated will look like:

    <a class="link-sort" href="http://www.example.com/books?field=id&amp;sort=desc">
        ID <i class="fa fa-sort"></i>
    </a>

#####What about if we want to combine this all together, with pagination and sorting? Easy:

In your controller:

    $books = Book::sort(Input::get('field'), Input::get('sort'))->paginate(25);
    
    return view('books.index', compact('books'));

In your view:

    {{ $books->columns(array(
                'id' => 'ID',
                'title' => 'Title',
                'author' => 'Authored By',
                'owned_by' => 'Owned By',
                'publisher' => 'Publisher',
            ))
            ->means('owned_by', 'user')
            ->modify('owned_by', function($user, $book) {
                return $user->first_name . ' ' . $user->last_name;
            })
            ->means('publisher', 'publisher')
            ->modify('publisher', function($publisher, $book) {
                return 'The publisher of this book: '. $publisher->name;
            })
            ->sortable(array('id', 'title'))
            ->showPages()
            ->render()
    }}

#####What if I want to generate a table for a relationship?:

In your controller:

    $book = Book::with('authors')->find(1);
    
    return view('book.show', compact('book'));

In this case, the book is going to have many authors (`hasMany` relationship)

In your view:

    {{ $book->authors->columns(
            'id' => 'ID',
            'name' => 'Name',
            'books' => 'Total # of Books'
        )
        ->means('books', 'num_of_books')
        ->render()
    }}

Keep in mind, we cannot paginate the table, or provide sortable columns on relationships. If you need this, grab it separately:

In your controller:

    $book = Book::find(1);

    $authors = Authors::where('book_id', $book->id)->paginate(25);

    return view('books.show', array(
        'book' => $book,
        'authors' => $authors,
    ));

In your view:

    {{ 
        $authors->columns(array(
            'name' => 'Name',
        ))->render()
    }}

#####Customizing table attributes using `attributes($attributes = array())`

    {{ 
        $authors->columns(array(
            'name' => 'Name',
        ))
        ->attributes(array(
            'id' => 'table-1',
            'class' => 'table table-striped table-bordered',
        ))
        ->render()
    }}

#####Showing your pages somewhere else:

Just don't call the `showPages()` method on the collection and put your pages
somewhere on your page like you would regularly do.

    {{ 
        $authors->columns(array(
            'name' => 'Name',
        ))
        ->attributes(array(
            'id' => 'table-1',
            'class' => 'table table-striped table-bordered',
        ))
        ->render()
    }}

    <div class="text-center">{{ $authors->appends(Input::except('page'))->links() }}</div>

##TO DO 

Relationship sorting