# Post Models

Post Models are the bread and butter of Fresa. They allow you to take an ordinary WordPress post type and treat it like a first-class object-oriented citizen.

To use Fresa's Post Model interface, create a new subclass of `Fresa\PostModel`. For this example, we will assume you are building an events plugin.

    use Fresa\PostModel;

    class Event extends PostModel{}

Once your subclass is defined, you can begin interacting with the existing WordPress data model right away.

## Post Types

If you don't define a custom post type, Fresa will use the snake-case version of the class name. If you want to define a custom post type slug, update the `$postType` property:

    use Fresa\PostModel;

    class Event extends PostModel
    {
        // Optionally define the post type
        protected $postType = 'custom_event';
    }

## Accessing Default Properties

The WordPress post object is accessible in a more convenient form:

    $event->title;   // post_title
    $event->content; // post_content
    $event->date;    // post_date
    $event->author;  // post_author

### Available Default Post Properties

A good rule of thumb is to access WordPress properties without the `post_` prefix.

WordPress Property | Fresa Equivalent
--------|-------
`ID` | `id`
`post_title` | `title`
`post_content` | `content`
`post_author` | `author`
`post_date` | `date`
`post_status` | `status`

## Instantiating Models

Models can be instantiated like standard PHP classes, or you can pass an array of initial arguments to populate the instance:

    // Instantiate with no arguments
    $event = new Event;
    $event->title = "My fun event";

    // Instantiate with initial arguments
    $event = new Event([
        'title' => "My fun event"
    ]);
    $event->title; // "My fun event"

## Saving Models

Custom models can be instantiated and built without being persisted to the database right away:

    $event = new Event;
    $event->title = "My fun event";
    $event->content = "Come have fun with us!";

    $event->title; // "My fun event"
    $event->id; // 0

When you are ready to persist your model to the database, call the `save()` method on the model:

    $event->save();
    $event->id; // 123

You can instantiate and persist a model in one easy step using the static `create()` on your model:

    $event = Event::create([
        'title' => "My fun event"
    ]);

    $event->title; // "My fun event"
    $event->id; // 123

## Required Attributes

By default, no attributes are required on a `PostModel`. However, you may require an attribute to be set before a model can be saved. Update the `$required` property to do so:

    use Fresa\PostModel;

    class Event extends PostModel
    {
        protected $required = [
            'title',
            'start',
        ];
    }

    Event::create(); // Throws Exception

    Event::create([ // Throws Exception
        'title' => "My fun event",
    ]);

    Event::create([ // Success!
        'title' => "My fun event",
        'start' => "today",
    ]);

Note that both WordPress-default fields and meta fields can be marked as required.

### Validation

If you want to validate a model instance before attempting to save it, you can use the `validate` method on the model:

    $event = new Event([
        'title' => "My fun event",
    ]);
    $event->validate(); // false

    $event->start = 'today';
    $event->validate(); // true

### Post Status

The default post status for new models is `publish`. You can change the default post status by overriding the `getDefaultStatus()` method on your model:

    public function getDefaultStatus()
    {
        return 'draft';
    }

## Deleting Models

To delete models, use the `delete` method:

    $event->delete();

## Custom Post Type Registration

In order to interface with a custom post type in the WordPress admin interface, you need to register it. With Fresa, you can register a custom post type with a single line of code:

    Event::register();

When using the `register()` method, Fresa requires you to have a custom `$postType` variable set.

By default, Fresa will use the name of the class as the post type label. If you'd like to customize the post type label, which is used throughout the WordPress admin, set `$postTypeName` to a string:

    class Event extends PostModel
    {
        protected $postType = 'event';
        protected $postTypeName = 'Company Event';
    }

You can override any of the options or labels registered with the post type as an argument to the method:

    Event::register([
        'supports' => ['title', 'editor', 'thumbnail'],
        'show_in_nav_menus' => false,
    ]);

## Helpers

Fresa provides various helper methods on `PostModel` objects which are shortcuts to existing WordPress functionality:

    $event->permalink(); // https://foo.bar/baz
    // Same as:
    get_permalink($event->ID);

>**Note**: More helpers will be added in the future. In the meantime, feel free to add your own helpers to your child class. Submit a pull request if you think it would help others!
