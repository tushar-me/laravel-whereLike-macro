`AppServiceProvider.php`

```ruby
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Support\Arr;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Builder::macro('whereLike', function ($attributes, string $searchTerm) {
            return $this->where(function (Builder $query) use ($attributes, $searchTerm) {
                foreach (Arr::wrap($attributes) as $attribute) {
                    $query->when(
                        !($attribute instanceof \Illuminate\Contracts\Database\Query\Expression) &&
                        str_contains((string) $attribute, '.'),
                        function (Builder $query) use ($attribute, $searchTerm) {
                            [$relation, $relatedAttribute] = explode('.', (string)$attribute, 2);
                            $query->orWhereRelation($relation, $relatedAttribute, 'LIKE', "%{$searchTerm}%");
                        },
                        function (Builder $query) use ($attribute, $searchTerm) {
                            $query->orWhere($attribute, 'LIKE', "%{$searchTerm}%");
                        }
                    );
                }
            });
        });
    }
}
```


## Example Usage:

`PostController.php`

```ruby
  use App\Models\Post;

  $posts = Post::query()
      ->whereLike([
          'title',                       
          'description',                 
          'user.name',         
          'comments.user.email',
          DB::raw('DATE_FORMAT(created_at, "%d/%m/%Y")'), 
      ], request()->get('search', ''))
      ->with(['user', 'comments.user']) 
      ->get();
```
