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
    
 3.    
