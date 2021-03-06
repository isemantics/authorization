## Policies

Policies are classes that resolve permissions for a given object. You can create
policies for any class in your application that you wish to apply permissions
checks to.

### Creating Policies

You can create policies in your `src/Policy` directory. Policy classes for ORM
objects follow the following conventions.

### Policy conventions for ORM classes

1. Policy classes use the `Policy` suffix.
2. Policy classes live in the `App\Policy` namespace or `$plugin\Policy`.

The default ORM resolver uses the following conventions for policy classes:

1. The entity classname is used to generate the policy class name. e.g
   `App\Model\Entity\Article` will map to `App\Policy\ArticlePolicy`.
2. Plugin entities will first check for an application policy e.g
   `App\Policy\Blog\ArticlePolicy` for `Blog\Model\Entity\Article`.
3. If no application override policy can be found, a plugin policy will be
   checked. e.g. `Blog\Policy\ArticlePolicy`.
4. If no policy can be found an exception will be raised.

For now we'll create a policy class for the `Article` entity in our application.
In **src/Policy/ArticlePolicy.php** put the following content:

```php
<?php
namespace App\Policy;

use App\Model\Entity\Article;
use Authorization\IdentityInterface;

class ArticlePolicy
{
}
```

### Writing Policy Methods

The policy class we just created doesn't do much right now. Lets define a method
that allows us to check if a user can update an article.

```php
public function canUpdate(IdentityInterface $user, Article $article)
{
    return $user->id == $article->user_id;
}
```

Policy methods must return `true` to indicate success. All other values will be
interpreted as failure.

### Policy Scopes

In addition to policies being able to define pass/fail authorization checks,
they can also define 'scopes'. Scope methods allow you to modify another object
applying authorization conditions. A perfect use case for this is restricting
a list view to the current user:

```php
namespace App\Policy;

class ArticlesPolicy
{
    public function scopeIndex($user, $query)
    {
        return $query->where(['Articles.user_id' => $user->getIdentifier()]);
    }
}
```


### Policy Preconditions

In some policies you may wish to apply common checks across all operations in
a policy. This is useful when you need to deny all actions to the provided
resource. To use preconditions you need to implement the `BeforePolicyInterface`
in your policy:

```php
namespace App\Policy;

use Authorization\Policy\BeforePolicyInterface;

class ArticlesPolicy implements BeforePolicyInterface
{
    public function before($user, $resource, $action)
    {
        if ($user->getOriginalData()->is_admin) {
            return true;
        }
        // fall through
    }
}
```

Before hooks are expected to one of 3 return values:

- `true` The user is allowed to proceed with the action.
- `false` The user is not allowed to proceed with the action.
- `null` The before hook did not make a decision, and the authorization method
  will be invoked.
