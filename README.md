# flask-note-9-Pagination
内容+图片太多，用分页减少内存压力

1.routes.py/ home
  首先，把json数据放进原有的结构中，创建好多个posts
  
  先见一个json文件，然后用get获取数据
  
      # add new dummy posts
      with open('posts.json') as f:
          data = json.load(f)

      for x in data:
          post = Post(title=x.get('title'), content=x.get('content'), user_id=x.get('user_id')) 
          db.session.add(post)
      db.session.commit()
      # end new dummy json crying 
      
              ⬆️⬆️⬆️
      ///////// 有问题  ////////（补一下json的添加+删除）
      
      page=request.args.get('page',1, type=int)
      posts = Post.query.paginate(page=page, per_page=2)
      
      设置分页，具体操作+原理见下一个section：
  
2. 在terminal里面看一下pagination的用法：
    
    2.1 默认20  per-page
    2.2 定义posts=all
    2.3 先找页数 posts.page
    2.4 打印当前页 for post in posts.items:
    2.5 posts=分页定义  +  改变每页项目数posts = Post.query.paginate(per_page=7) 
    2.6 分好页后找到页面 posts = Post.query.paginate(per_page=7, page=7)
    2.7 看总数 posts.total 
    
              先进入python3
              >>> from flaskblog.models import Post
             
              >>> posts = Post.query.all() 定义posts=所有的posts数据，然后开始写逻辑
              >>> for post in posts:  
              ...     print(post)  在复数项里面的单个项，把它们打印出来
              ... 
              <Post 1>
                 ～
              <Post 74>
              >>> posts = Post.query.paginate() 现在把所有posts设为分页
              >>> posts
              <flask_sqlalchemy.Pagination object at 0x110c2fcd0>
              >>> dir(posts)    用个dir干什么呢？？？？
              ['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'has_next', 'has_prev', 'items', 'iter_pages', 'next', 'next_num', 'page', 'pages', 'per_page', 'prev', 'prev_num', 'query', 'total']
              >>> posts.per_page    默认20项每页。
              20
              >>> posts.page        当前页面
              1
              >>> for post in posts.items: 当前页的所有，打出来
              ...     print(post)
              ... 
              <Post 1>
               ～
              <Post 20>
              >>> posts = Post.query.paginate(page=2) 分页当中的第二页打印出来
              >>> for post in posts.items:
              ...     print(post)
              ... 
              <Post 21>
                ～
              <Post 40>
              >>> posts = Post.query.paginate(per_page=7) 接下来，定义每页7个项目！per_page=7，并打印当前界面
              >>> posts.page
              1
              >>> for post in posts.items:
              ...     print(post)
              ... 
              <Post 1>
                ～
              <Post 7>
              >>> posts = Post.query.paginate(per_page=7, page=7)
              >>> posts.page      想去到每页为7时候的第7页。要写一起：(per_page=7, page=7)
              7
              >>> for post in posts.items:
              ...     print(post)
              ... 
              <Post 43>
                ～
              <Post 49>
              >>> posts.total   查看总数
              74
              >>> 
    
 3.  页数设计
 home.html
 
        {% for post in posts.items %}改完分页，顺便在这里也改一下

        {% endfor %}
        {% for page_num in posts.iter_pages(left_edge=1, right_edge=1, left_current=1, right_current=2) %}  左右各留多少页书
          {% if page_num%}
            {% if posts.page == page_num %}     如果是当前页面，则变成实心按钮
              <a class="btn btn-info mb-4" href="{{ url_for('home', page=page_num)}}">{{ page_num }}</a>
            {% else %}      否则是空心蓝色默认按钮
              <a class="btn btn-outline-info mb-4" href="{{ url_for('home', page=page_num)}}">{{ page_num }}</a>
            {% endif %}
          {% else %}      要不然就是省略号代替
            ...
          {% endif %}
        {% endfor %}

4. 新旧展示顺序
home.html加一个用顺序过滤

        posts = Post.query.order_by(Post.date_posted.desc()).paginate(page=page, per_page=10)
        

5. 点击用户，进入他自己的所有posts

      5.1 建一个新的路径：
      
        @app.route("/user/<string:username>")
        def user_posts(username):
            page=request.args.get('page',1, type=int)
            user = User.query.filter_by(username=username).first_or_404() 把用户名打上去
            posts = Post.query.filter_by(author=user)\    通过名字筛选
                .order_by(Post.date_posted.desc())\     倒序
                .paginate(page=page, per_page=10)     分页
            return render_template('user_posts.html', posts=posts, user=user)
 
      5.2 页面内部和home很像：
      用户界面基本和home一样，但是有几个地方需要修改：
      
        {% extends "layout.html" %}
        {% block content %}
            <h1 class="mb-3">Posts by {{ user.username }}({{ posts.total }})</h1> 加了一个标题，用户名+数量
            {% for post in posts.items %}
            <article class="media content-section">
              <img src="{{ url_for('static', filename='profile_pics/' + post.author.image_file) }}" class="rounded-circle article-img">
                <div class="media-body">
                  <!-- <div class="article-metadata"> -->
                    <a class="mr-2" href="{{ url_for('user_posts', username=post.author.username) }}">{{ post.author.username }}</a>                这里，引用引的是名字，导航也是这一页。 在/home + /post 见面也都要修改
                    <small class="text-muted">{{ post.date_posted.strftime('%Y-%m-%d') }}</small>
                  <!-- </div> -->
                  <h2><a class="article-title" href="{{ url_for('post', post_id=post.id) }}">{{ post.title }}</a></h2>
                  <p class="article-content">{{ post.content }}</p>
                </div>
            </article>
            {% endfor %}
        
