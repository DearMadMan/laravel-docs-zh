# Eloquent: 关联

- [前言](#introduction)
- [定义关联](#defining-relationships)
    - [一对一](#one-to-one)
    - [一对多](#one-to-many)
    - [一对多（反向）](#one-to-many-inverse)
    - [多对多](#many-to-many)
    - [远程一对多](#has-many-through)
    - [多态关联](#polymorphic-relations)
    - [多对多多态关联](#many-to-many-polymorphic-relations)
- [关联查询](#querying-relations)
    - [关联查询方法 Vs. 动态属性](#relationship-methods-vs-dynamic-properties)
    - [查询存在的关联](#querying-relationship-existence)
    - [度量关联模型](#counting-related-models)
- [预加载](#eager-loading)
    - [预加载约束](#constraining-eager-loads)
    - [懒加载](#lazy-eager-loading)
- [插入 & 更新关联模型](#inserting-and-updating-related-models)
    - [`save` 方法](#the-save-method)
    - [`create` 方法](#the-create-method)
    - [Belongs To 关联](#updating-belongs-to-relationships)
    - [多对多关联](#updating-many-to-many-relationships)
- [时间戳](#touching-parent-timestamps)

<a name="introduction"></a>
## 前言

数据库中的表经常性的与其它的表相关联。比如，一个博客文章可以有很多的评论，或者一个订单会关联一个下单的用户。Eloquent 使管理和协作这些关系变的非常的容易，并且支持多种不同类型的关联：

- [一对一](#one-to-one)
- [一对多](#one-to-many)
- [多对多](#many-to-many)
- [远程一对多](#has-many-through)
- [多态关联](#polymorphic-relations)
- [多对多多态关联](#many-to-many-polymorphic-relations)

<a name="defining-relationships"></a>
## 定义关联

Eloquent 关联可以像定义方法一样在 Eloquent 模型类中进行定义。同时，它就像 Eloquent 模型自身一样也提供了强大的 [查询生成器](/{{language}}/{{version}}/queries)。这允许关联模型可以链式的执行查询能力。比如，我们可以在 `posts` 关联中添加额外的约束：

    $user->posts()->where('active', 1)->get();

但是，在更深入的使用关联之前，让我们先来学习一下如何定义各种类型的关联。

<a name="one-to-one"></a>
### 一对一

一对一的关联是最基础的关联。比如，一个 `User` 模型可能关联一个 `Phone`。我们需要在 `User` 模型上放置一个 `phone` 方法来定义这种关联。`phone` 方法应该返回一个 `hasOne` 方法的结果：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the phone record associated with the user.
         */
        public function phone()
        {
            return $this->hasOne('App\Phone');
        }
    }

传递到 `hasOne` 方法的第一个参数应该是关联模型的名称。一旦关联被定义完成，我们可以使用 Eloquent 的动态属性来访问关联模型的记录。动态属性允许你访问关联函数，就像是它们是定义在模型中的属性一样：

    $phone = User::find(1)->phone;

Eloquent 假定所关联的外键是基于模型的名称的。在这个前提下，`Phone` 模型会自动的假定其拥有一个 `user_id` 外键。如果你希望修改这个惯例，你可以传递第二个参数到 `hasOne` 方法中：

    return $this->hasOne('App\Phone', 'foreign_key');

另外，Eloquent 也会假定外键应该在其上层模型上拥有一个匹配的 `id`（或者自定义的 `$primaryKey`）值。换句话说，Eloquent 会查询 `Phone` 记录中的 `user_id` 列所对应的用户的 `id` 列的记录。如果你希望关联使用 `id` 以外的值，你可以传递第三个参数到 `hasOne` 方法来指定自定义的键：

    return $this->hasOne('App\Phone', 'foreign_key', 'local_key');

#### 定义相对的关联

那么，我们可以从我们的 `User` 中访问 `Phone` 模型。现在，让我们在 `Phone` 模型上定义一个关联，着让我们可以从 `Phone` 模型中访问其所属的 `User`。我们使用 `belongsTo` 方法来定义 `hasOne` 相对的关联：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Phone extends Model
    {
        /**
         * Get the user that owns the phone.
         */
        public function user()
        {
            return $this->belongsTo('App\User');
        }
    }

在上面的例子中，Eloquent 将会尝试从 `Phone` 模型中的 `user_id` 字段中匹配查找 `id` 相同的 `User`。Eloquent 会依据所关联的模型的蛇形命名和 `_id` 后缀来假定默认的外键名。事实上，如果在 `Phone` 模型上的外键不是 `user_id`，那么你可以传递自定义的外键名到 `belongsTo` 方法的第二个参数：

    /**
     * Get the user that owns the phone.
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key');
    }

如果你的上级模型并没有使用 `id` 作为主键名，或者你希望下级模型关联一个不同的列。你可以传递第三个参数到 `belongsTo` 方法来指定上级模型表中的自定义键：

    /**
     * Get the user that owns the phone.
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key', 'other_key');
    }

<a name="one-to-many"></a>
### 一对多

一个一对多的关联常常用来定义一个模型拥有其他任意数目的模型。比如，一个博客文章可以拥有很多条评论。就像其他的 Eloquent 关联一样，一对多关联在 Eloquent 模型中通过方法来进行定义：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * Get the comments for the blog post.
         */
        public function comments()
        {
            return $this->hasMany('App\Comment');
        }
    }

记住，Eloquent 会自动的根据 `Comment` 模型来判断合适的外键。依据惯例，Eloquent 会使用自身模型的蛇形命名和 `_id` 来作为外键。所以，在这个例子中，Eloquent 会假定 `Comment` 模型的外键是 `post_id`。

一旦关联定义完成之后，我们可以通过 `comments` 属性来访问所有关联的评论的集合。记住，由于 Eloquent 提供了动态属性，我们可以对关联函数进行访问，就像他们是在模型中定义的属性一样：

    $comments = App\Post::find(1)->comments;

    foreach ($comments as $comment) {
        //
    }

当然，由于所有的关联都提供了查询生成器的功能，所以你可以在调用 `comments` 方法时继续的添加一些限制条件，你可以通过链式的调用来进行查询条件的添加：

    $comments = App\Post::find(1)->comments()->where('title', 'foo')->first();

就像 `hasOne` 方法，你可以通过添加额外的参数到 `hasMany` 方法中来重置外键和主键：

    return $this->hasMany('App\Comment', 'foreign_key');

    return $this->hasMany('App\Comment', 'foreign_key', 'local_key');

<a name="one-to-many-inverse"></a>
### 一对多（反向）

现在我们可以访问文章中所有的评论了，让我们为评论定义一个关联使其可以访问它的上层文章模型。为了定义一个 `hasMany` 相对的关联，你需要在下层模型中定义一个关联方法并调用 `belongsTo` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * Get the post that owns the comment.
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

一旦关联被定义完成，我们就可以通过 `Comment` 模型的 `post` 动态属性来检索到其对应的 `Post` 模型：

    $comment = App\Comment::find(1);

    echo $comment->post->title;

在上面的例子中，Eloquent 会尝试从 `Comment` 模型中的 `post_id` 字段检索与其相对应 `id` 的 `Post` 模型。Eloquent 会使用关联模型的蛇形命名和 `_id` 后缀来作为默认的外键。如果 `Comment` 模型的外键不是 `post_id`，你可以传递一个自定义的键名到 `belongsTo` 方法的第二个参数：

    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key');
    }

如果上层模型并没有使用 `id` 作为主键，或者你想在下层模型中关联其他的列，你可以传递第三个参数到 `belongsTo` 方法中：

    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key', 'other_key');
    }

<a name="many-to-many"></a>
### 多对多

多对多的关联比 `hasOne` 和 `hasMany` 关联要稍微复杂一些。假如一个用户拥有多个角色，而角色又可以被其他的用户所共享。比如，多个用户可以拥有管理员的角色。如果定义这种关联，我们需要定义三个数据库表：`users`，`roles`，和 `role_user`。`role_user` 表的命名是以相关联的两个模型数据表来依照字母顺序命名，并且表中包含了 `user_id` 和 `role_id` 列。

多对多关联需要编写一个方法调用 `belongsToMany` 方法。比如，让我们在 `User` 模型中定义一个 `roles` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The roles that belong to the user.
         */
        public function roles()
        {
            return $this->belongsToMany('App\Role');
        }
    }

一旦关联被定义，你可以通过 `roles` 动态属性来访问用户的角色：

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        //
    }

当然，就像其他类型的关联，你也可以调用 `roles` 方法进而链式添加查询条件：

    $roles = App\User::find(1)->roles()->orderBy('name')->get();

就如先前所提到的，Eloquent 会合并两个关联模型并依照字母顺序进行命名表名。当然你也可以随意的重写这个约定，你可以传递第二个参数到 `belongsToMany` 方法：

    return $this->belongsToMany('App\Role', 'role_user');

除了自定义合并数据表的名称之外，你也可以通过往 `belongsToMany` 方法传传递额外参数来自定义数据表里的键的字段名称。第三个参数是你定义在关联中模型外键的名称。第四个参数则是你要合并的模型外键的名称：

    return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');

#### 定义相对的关联

你只需要在相对应的关联模型里放置其他的方法来调用 `belongsToMany` 方法就可以定义相对关联。继续我们上面的用户角色示例，让我们在 `Role` 模型中定义一个 `users` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\User');
        }
    }

就如你所看到的，这个关联的定义与用 `App\User` 模型的关联定义完全相同。因为我们重复的使用了 `belongsToMany` 方法，当定义相对多对多的关联时，所有常用的自定义数据表和键的选项都是可用的。

#### 检索中间表的列

正如你已经了解到的。定义多对多的关联需要引入一个中间表。Eloquent 提供了几种非常有帮助的方式来与这个表进行交互。比如，让我们假定我们的 `User` 对象关联到了很多 `Role` 对象。在访问这些关联对象时，我们可以通过在模型上使用 `pivot` 属性来访问中间表：

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }

注意我们取出的每个 `Role` 对象，都会被自动的分配 `pivot` 属性。这个属性包含了一个代表中间表的模型，并且可以像其他 Eloquent 模型一样被使用。

默认的，只有模型的键会被 `pivot` 对象提供，如果你的中间表包含了额外的属性，你必须在定义关联时指定它们：

    return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');

如果你想要中间表自动维护 `created_at` 和 `updated_at` 时间戳，你可以在定义关联时使用 `withTimestamps` 方法：

    return $this->belongsToMany('App\Role')->withTimestamps();

#### 通过中间表字段过滤关联

你可以通过在定义关联时使用 `wherePrivot` 和 `wherePivotIn` 方法来在返回的结果中进行过滤：

    return $this->belongsToMany('App\Role')->wherePivot('approved', 1);

    return $this->belongsToMany('App\Role')->wherePivotIn('priority', [1, 2]);

<a name="has-many-through"></a>
### 远程一对多

远程一对多关联提供了简短便捷的方法通过中间关联件来访问远端的关联。比如，一个 `Country` 模型应该通过 `User` 模型可以拥有很多的 `Post` 模型。在这个例子中，你可以非常容易的就检索出一个国家中的所有的文章。让我们来看一下定义这些关联所需要的表:

    countries
        id - integer
        name - string

    users
        id - integer
        country_id - integer
        name - string

    posts
        id - integer
        user_id - integer
        title - string

远端的 `posts` 并没有包含 `country_id` 列，`hasManyThrough` 关联可以通过 `$country->posts` 来访问一个国家的文章。为了执行这个查询，Eloquent 会通过中间表 `users` 的 `country_id` 来检索 `posts` 表中用户 ID 相匹配的记录。

现在我们已经明确了关联表的结构，那么让我们来在 `Country` 模型上定义关联：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Country extends Model
    {
        /**
         * Get all of the posts for the country.
         */
        public function posts()
        {
            return $this->hasManyThrough('App\Post', 'App\User');
        }
    }

传递到 `hasManyThrough` 方法的第一个参数是我们最终想要访问到的模型，而第二个参数则是中间层的模型名称。

当使用关联查询时，通常 Eloquent 会遵循外键约定。如果你希望对关联的键进行自定义，你可以传递第三和第四个参数到 `hasManyThrough` 方法。第三个参数是中间层模型的外键名称，第四个参数是最终想要获取的模型中的所对应的中间层的外键, 而第五个参数则是当前模型的主键：

    class Country extends Model
    {
        public function posts()
        {
            return $this->hasManyThrough(
                'App\Post', 'App\User',
                'country_id', 'user_id', 'id'
            );
        }
    }

<a name="polymorphic-relations"></a>
### 多态关联

#### 表结构

多态关联允许一个模型在单个关联中从属一个或多个其它模型。比如，想象一下应用中的用户可以对文章及视频进行评论。如果使用多态关联，那么你就可以使用一个单独的 `comments` 表来关联这两个场景。首先，让我们确定定义这种关联所需要的表结构：

    posts
        id - integer
        title - string
        body - text

    videos
        id - integer
        title - string
        url - string

    comments
        id - integer
        body - text
        commentable_id - integer
        commentable_type - string

你需要注意到的两个在 `comments` 表中重要的字段 `commentable_id` 和 `commentable_type`。`commentable_id` 字段会包含文章或者视频的 ID 值，而 `commentable_type` 字段会包含其所属的模型的类名。`commentable_type` 就是当访问 `commentable` 关联时 ORM 用来判断所属的模型是哪个类型。

#### 模型结构

接着，让我们检查一下这个关联所需要的模型定义：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * Get all of the owning commentable models.
         */
        public function commentable()
        {
            return $this->morphTo();
        }
    }

    class Post extends Model
    {
        /**
         * Get all of the post's comments.
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

    class Video extends Model
    {
        /**
         * Get all of the video's comments.
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

#### 获取多态关联

一旦数据库表和模型都定义完成，你就可以在你的模型中访问这些关联。比如，你可以使用 `comments` 动态属性来访问文章中所有关联的 comments 模型:

    $post = App\Post::find(1);

    foreach ($post->comments as $comment) {
        //
    }

你也可以通过在模型上调用提供 `morphTo` 的方法来获取多态模型其关系所有者。在上面的例子中，指的就是 `Comment` 模型中的 `commentable` 方法。所以，我们可以像使用动态属性一样使用方法来进行访问：

    $comment = App\Comment::find(1);

    $commentable = $comment->commentable;

`Comment` 模型的 `commentable` 关联将会返回一个 `Post` 或者 `Video` 实例，这取决于其所属者的类型。

#### 自定义多态类型

默认的，Laravel 会使用完全限定类名来存储所关联模型的类型。比如，上面的例子中 `Comment` 可以属于 `Post` 或者 `Video`。默认的 `commentable_type` 应该是 `App\Post` 或者 `App\Video`。事实上，你可能希望从你的应用程序的内部结构分离数据库。在这个例子中，你可以定义一个关联的多态映射来指导 Eloquent 使用模型关联的表名称来替代类名：

    use Illuminate\Database\Eloquent\Relations\Relation;

    Relation::morphMap([
        'posts' => App\Post::class,
        'videos' => App\Video::class,
    ]);

你可以在你的 `AppServiceProvider` 或者一个分离的服务提供者的 `boot` 方法中注册你的 `morphMap`。

<a name="many-to-many-polymorphic-relations"></a>
### 多对多多态关联

#### 表结构

除了传统的多态关联，你也可以定义多对多的多态关联。比如，一个博客的 `Post` 和 `Video` 模型应该可以共享一个多态关联的 `Tag` 模型。使用多对多的多态关联可以允许你的博客文章和视频能够共享独特标签的单个列表。首先，让我们来看一下表结构：

    posts
        id - integer
        name - string

    videos
        id - integer
        name - string

    tags
        id - integer
        name - string

    taggables
        tag_id - integer
        taggable_id - integer
        taggable_type - string

#### 模型结构

接着，我们来定义模型中的关联。`Post` 和 `Video` 模型将都会包含调用基础 Eloquent 类的 `morphToMany` 方法的 `tags` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * Get all of the tags for the post.
         */
        public function tags()
        {
            return $this->morphToMany('App\Tag', 'taggable');
        }
    }

#### 定义相对的关联

接着，在 `Tag` 模型中，你应该为所有关联模型定义相应的方法。所以，在这个例子中，我们将定义 `posts` 方法和 `videos` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Tag extends Model
    {
        /**
         * Get all of the posts that are assigned this tag.
         */
        public function posts()
        {
            return $this->morphedByMany('App\Post', 'taggable');
        }

        /**
         * Get all of the videos that are assigned this tag.
         */
        public function videos()
        {
            return $this->morphedByMany('App\Video', 'taggable');
        }
    }

#### 获取关联

当定义完成数据表和模型之后，你就可以通过模型来访问其关联。比如，你可以简单的使用 `tags` 动态属性来访问当前文章所有的标签模型：

    $post = App\Post::find(1);

    foreach ($post->tags as $tag) {
        //
    }

你也可以通过访问模型中提供执行 `morphedByMany` 方法的方法来获取关联模型的所属模型。在上面的例子中，就是 `Tag` 模型上的 `posts` 或者 `videos` 方法。所以，你可以像动态属性一样访问这些方法：

    $tag = App\Tag::find(1);

    foreach ($tag->videos as $video) {
        //
    }

<a name="querying-relations"></a>
## 关联查询

由于所有的 Eloquent 关联类型都是通过方法定义的，所以你可以调用这些方法来获取所关联的模型的实例而无需实际的执行关联查询。另外，所有的 Eloquent 关联也都提供了 [查询生成器](/{{language}}/{{version}}/queries) 服务，这允许你可以继续的链式执行查询操作。

比如，想象一下博客系统中 `User` 模型拥有很多 `Post` 关联的模型：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get all of the posts for the user.
         */
        public function posts()
        {
            return $this->hasMany('App\Post');
        }
    }

你可以查询 `posts` 关联的同时添加一些额外的查询约束：

    $user = App\User::find(1);

    $user->posts()->where('active', 1)->get();

你应该注意到了，你可以在关联中使用任何的 [查询生成器](/{{language}}/{{version}}/queries) 的方法，所以，请探索查询生成器的文档来学习更多可用的方法。

<a name="relationship-methods-vs-dynamic-properties"></a>
### 关联方法 Vs. 动态属性

如果你不需要在进行 Eloquent 关联查询时添加额外的约束，你可以简单的像它的属性一样进行访问。比如，我们继续使用 `User` 和 `Post` 示例模型。我们可以像这样来访问用户的所有文章：

    $user = App\User::find(1);

    foreach ($user->posts as $post) {
        //
    }

动态属性是惰性加载的，这意味着在你实际访问他们之前，其关联数据是不会加载的。正因为如此，开发的时候通常使用预加载来进行加载一些即将用到的关联模型。[预加载](#eager-loading) 要求必须加载一个模型的关系，这有效的减少了查询的次数。

<a name="querying-relationship-existence"></a>
### 查询存在的关联

当访问一个模型的记录时，你可能会希望基于关联的记录是否存在来对结果进行限制。比如，想象一下你希望获取最少有一条评论的博客文章。你可以传递关联的名称到 `has` 方法来做这些：

    // Retrieve all posts that have at least one comment...
    $posts = App\Post::has('comments')->get();

你也可以指定操作符，和数量来进一步定制查询：

    // Retrieve all posts that have three or more comments...
    $posts = Post::has('comments', '>=', 3)->get();

你也可以使用 `.` 语法来构造嵌套的 `has` 语句。比如，你可以获取所有包含至少一条评论和投票的文章：

    // Retrieve all posts that have at least one comment with votes...
    $posts = Post::has('comments.votes')->get();

如果你需要更高的控制，你可以使用 `whereHas` 和 `orWhereHas` 方法来在 `has` 查询中插入 `where` 子句。这些方法允许你为关联进行自定义的约束查询。比如检查评论的内容：

    // Retrieve all posts with at least one comment containing words like foo%
    $posts = Post::whereHas('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

<a name="counting-related-models"></a>
### 度量关联模型

如果你希望统计关联的结果而不实际的加载它们，你可以使用 `withCount` 方法，这将在你的结果模型中添加 `{relation}_count` 列。比如：

    $posts = App\Post::withCount('comments')->get();

    foreach ($posts as $post) {
        echo $post->comments_count;
    }

你也可以同时检索多个关联的统计，以及添加查询约束：

    $posts = Post::withCount(['votes', 'comments' => function ($query) {
        $query->where('content', 'like', 'foo%');
    }])->get();

    echo $posts[0]->votes_count;
    echo $posts[0]->comments_count;

<a name="eager-loading"></a>
## 预加载

当通过属性访问 Eloquent 关联时，该关联的数据会被延迟加载。这意味着该关联数据只有在你真实的访问属性时才会进行加载。事实上，Eloquent 可以在上层模型中一次性预加载的。预加载有效避免了 N + 1 的查找问题。要说明 N + 1 查找问题，我们可以来看一个 `Author` 关联 `Book` 的示例：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Book extends Model
    {
        /**
         * Get the author that wrote the book.
         */
        public function author()
        {
            return $this->belongsTo('App\Author');
        }
    }

现在，让我们检索所有的书籍和他们的作者：

    $books = App\Book::all();

    foreach ($books as $book) {
        echo $book->author->name;
    }

这个循环会执行一次查找回所有的书籍，接着每本书会运行一次查找作者的操作。所以，如果我们拥有 25 本书，那么循环将会进行 26 次查询：1 次查询所有的书籍，25 次查询相关书籍的作者。

非常幸运的，我们可以使用预加载来将查询有效的控制在 2 次。当查询时，使用 `with` 方法来指定关联的预加载：

    $books = App\Book::with('author')->get();

    foreach ($books as $book) {
        echo $book->author->name;
    }

对于这个操作，只会执行两个查询：

    select * from books

    select * from authors where id in (1, 2, 3, 4, 5, ...)

#### 预加载多个关联

有时候你可能需要在一个操作中预加载多个不同的关联，你只需要传递额外的参数到 `with` 方法中就可以：

    $books = App\Book::with('author', 'publisher')->get();

#### 嵌套的预加载

你可以使用 `.` 语法来加载嵌套的关联。比如，让我们在一个 Eloquent 语句中一次加载所有书籍的作者以及作者的私人通讯簿：

    $books = App\Book::with('author.contacts')->get();

<a name="constraining-eager-loads"></a>
### 预加载约束

有时候你可能希望预加载一些关联，但是也需要对预加载查询指定额外的约束，这里有个示例：

    $users = App\User::with(['posts' => function ($query) {
        $query->where('title', 'like', '%first%');
    }])->get();

在这个例子中，Eloquent 会只预加载文章的 `title` 列包含 `first` 单词的记录。当然，你也可以调用其他 [查询生成器](/{{language}}/{{version}}/queries) 可用的方法来进一步的定制预加载操作：

    $users = App\User::with(['posts' => function ($query) {
        $query->orderBy('created_at', 'desc');
    }])->get();

<a name="lazy-eager-loading"></a>
### 延迟预加载

有时候你可能需要在上层模型被获取后才预加载其关联。当你需要来动态决定是否加载关联模型时尤其有用:

    $books = App\Book::all();

    if ($someCondition) {
        $books->load('author', 'publisher');
    }

如果你需要对预加载做一些查询约束，你可以传递一个包含待加载的关联与 `Closure` 的键值对数组到 `load` 方法，数组值应该是一个 `Closure` 实例，并且它会接收查询实例作为参数：

    $books->load(['author' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }]);

<a name="inserting-and-updating-related-models"></a>
## 插入 & 更新关联模型

<a name="the-save-method"></a>
### Save 方法

Eloquent 提供了方便的方法来为模型添加一个关联。比如，也许你需要为 `Post` 模型新增一个 `Comment`。除了手动的设置 `Comment` 的 `post_id` 属性，你也可以直接在关联模型中调用 `save` 方法来插入 `Comment`:

    $comment = new App\Comment(['message' => 'A new comment.']);

    $post = App\Post::find(1);

    $post->comments()->save($comment);

注意上面我们并没有使用关联模型的动态属性的方式来访问 `comments`，而是使用 `comments` 方法的形式来获取关联模型的实例。`save` 方法会自动的添加相应的 `post_id` 值到新的 `Comment` 模型上。

如果你需要一次添加多个关联模型，你需要使用 `saveMany` 方法：

    $post = App\Post::find(1);

    $post->comments()->saveMany([
        new App\Comment(['message' => 'A new comment.']),
        new App\Comment(['message' => 'Another comment.']),
    ]);

<a name="the-create-method"></a>
### Create 方法

除了 `save` 和 `saveMany` 方法之外，你也可以使用 `create` 方法，它可以接收属性组成的数组，然后创建一个模型并且将其存储到数据库。这一次，`save` 和 `create` 方法的区别是 `save` 接收一个完整的 Eloquent 模型实例，而 `create` 接收的是一个原生的 PHP `array`:

    $post = App\Post::find(1);

    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);

在使用 `create` 方法之前，你应该确保已经阅读了属性的 [批量赋值文档](/{{language}}/{{version}}/eloquent#mass-assignment)。

<a name="updating-belongs-to-relationships"></a>
### Belongs To 关联

当更新一个 `belongsTo` 关联时，你应该使用 `associate` 方法。这个方法会在下层模型中设置外键：

    $account = App\Account::find(10);

    $user->account()->associate($account);

    $user->save();

当删除 `belongsTo` 关联时，你应该使用 `dissociate` 方法，该方法会重置模型所关联的外键为 `null`：

    $user->account()->dissociate();

    $user->save();

<a name="updating-many-to-many-relationships"></a>
### 多对多关联

#### 附加 / 抽离

Eloquent 提供了一些额外的帮助方法来更方便的管理关联模型。比如，让我们想象一下用户可以有很多角色并且角色可以有很多用户。你可以使用 `attach` 方法来附加一个角色到用户并且在中间表中加入这条记录：

    $user = App\User::find(1);

    $user->roles()->attach($roleId);

当附加关联到模型时，你也可以传递一个含有额外数据的数组来将其添加到中间表中：

    $user->roles()->attach($roleId, ['expires' => $expires]);

当然，有时候你可能需要从用户中删除一个角色。你可以使用 `detach` 方法来删除多对多关联的记录。`datech` 方法将从中间表中删除相应的记录。但是，除了中间表之外，其它两个模型的记录都还会被保留：

    // Detach a single role from the user...
    $user->roles()->detach($roleId);

    // Detach all roles from the user...
    $user->roles()->detach();

为了更加的便捷，`attach` 和 `detach` 也可以接收 IDs 所组成的数组作为输入：

    $user = App\User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([1 => ['expires' => $expires], 2, 3]);

#### 同步

你也可以使用 `sync` 方法来构建多对多的关联。`sync` 方法接收放置中间表 IDs 所组成的数组。任意 IDs 如果没有在所给定的数组中，那么其将会从中间表中进行删除。所以，在操作完成之后，只有存在于给定数组里的 IDs 才会存在于中间表中：

    $user->roles()->sync([1, 2, 3]);

你也可以同时传递额外的中间表数据：

    $user->roles()->sync([1 => ['expires' => true], 2, 3]);

如果你不想要剥离已经存在的 IDs，那么你可以使用 `syncWithoutDetaching` 方法：

    $user->roles()->syncWithoutDetaching([1, 2, 3]);

#### 在中间表中存储额外的数据

当使用多对多关联时，`save` 方法可以接收中间表属性所组成的额外数据数组作为第二个参数：

    App\User::find(1)->roles()->save($role, ['expires' => $expires]);

#### 更新中间表的记录

如果你需要更新中间表中存在的行，你可以使用 `updateExistingPivot` 方法，该方法会接收中间表的外键和一个包含待更新属性的数组：

    $user = App\User::find(1);

	$user->roles()->updateExistingPivot($roleId, $attributes);

<a name="touching-parent-timestamps"></a>
## 联动上层模型时间戳

当一个模型 `belongsTo` 或者 `belongsToMany` 另外一个模型时，比如 `Comment` 从属于 `Post`，这将对于下层模型更新时同时要求更新上层模型的时间戳时会很有帮助。比如，当 `Comment` 模型更新了，你想要自动的更新其所属的 `Post` 模型的 `updated_at` 时间戳。Eloquent 使之变的非常容易。你只需要在下层模型中添加一个 `touches` 属性来包含关联的名称就可以了：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * All of the relationships to be touched.
         *
         * @var array
         */
        protected $touches = ['post'];

        /**
         * Get the post that the comment belongs to.
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

现在，当你更新 `Comment` 时，其所属的 `Post` 将会同时更新 `updated_at` 列，这将方便的知道 `Post` 模型的缓存何时失效：

    $comment = App\Comment::find(1);

    $comment->text = 'Edit to this comment!';

    $comment->save();
