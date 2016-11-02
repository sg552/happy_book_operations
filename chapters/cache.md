# 缓存

我们做web开发， 如果一个页面逻辑非常复杂，打开需要： 访问数据库20次， 那么这个页面，
打开时间 需要 2800ms。 我们就可以判定，这个页面的打开时间过长，需要优化。

优化的方式： 不用每次都现从 数据库中读数据。 直接把上一次的结果， 显示出来就可以。

例如： 如果后台的数据， 一天都不变化。 (我的个人博客，一天都没有新文章出现）， 那么
客户，访问我的个人博客时， 这一天看到的内容都是一样的。

如果这是个常态的话，我就可以把我的博客的首页，设置一个缓存。 缓存的有效时间是一天。
那么，每次rails 刷新页面的时候，都不会读取数据库。

正常的逻辑：

请求 -> rails/php  -> database -> rails  -> 前台显示

缓存的逻辑：

  - 请求 -> rails/php  -> database -> rails -> 保存到缓存 -> 前台显示
  - 请求 -> rails -> 读缓存 -> 前台显示

缓存的作用是巨大的。

好的缓存， 可以减少90%左右的 请求。 极大的提升效率

## 缓存的要点

只有一点：  找到性能的瓶颈！ 在最应该做缓存的地方， 做缓存。

(我们会发现， 一个事儿， 有很多环节。 其实，哪个环节上都可以做缓存。 但是我们要在最明显的地方做缓存）

## 缓存，设计不好，是个双刃剑。

普通代码：

```
def show
  @book = Book.find params[:id]
end
```

缓存代码：


```
def show
  key = "book_#{params[:id]}"
  if cache(key).expired?
    @book = Book.find params[:id]
    cache(key) = @book
  else
    @book = cache(key)
  end
end
```

上面就是缓存的最基本概念。 也是最干的干货。

短板：  我们一旦用了缓存，就要无时不刻的考虑到它的过期，维护等等。 会让我们的代码极度膨胀。难以维护。

我们的一个原则： 只在不得不用的时候(严重的影响了性能）， 才用缓存。

## Rails中的缓存。

缓存在使用的时候， 都是要封装的。如果不封装，代码就很臃肿。

Rails帮我们做了几种封装， 常见的是： View层面的缓存。 和 Rails.cache这个基本层面的。

### page cache

https://github.com/rails/actionpack-page_caching

```
config.action_controller.page_cache_directory = "#{Rails.root.to_s}/public/deploy"
```

```
class WeblogController < ActionController::Base
  caches_page :show, :new
end
```

这个配置，高度封装了 基本的缓存操作。

其他的缓存， fragment， sql, action cache...都不建议使用。 难以驾驭。（封装的越是厉害， 调试就不好调试。
特别是缓存这个东西，跟薛定谔的猫一样， 跟当时的状态有关。

所以缓存特别不好调试，除非在特别必要的地方，我们才用“最简单“形式的缓存。
