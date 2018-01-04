# Your First Flask App

Flask is a Python web framework. In its simplest terms, it maps URLs that users can type in their browsers to Python functions.

In Your First Flask App, we'll be:

* Displaying HTML content;
* Allowing users to submit a form; and
* Saving things to an in-memory database.

The app takes the form of a simple blog application—where a user can add new posts to our blog.

## Our URLs

The app has three URLs that we'll be programming:

* `/`: the home, will show our blog homepage and a list of posts.
* `/post/<post_id>`: the post page, when given a `post_id` it will show the correct Post (given it exists).
* `/post/create`: the new post page, will show a form and allow users to submit it to create a new post.

Each of these routes is mapped to a Python function.

There is also a global variable which stores our blog and posts called `blog`. It is a dictionary with this format:

```python
blog = {
	'name': 'My awesome blog',
	'posts': {}
}
```

The `'posts'` key is another dictionary which will take the format `{id: post}`, like so:

```python
{
	1: {
		'id': 1,
		'title': 'My post',
		'content': 'My post content.'
	},
	2: {
		'id': 2,
		'title': 'Another post',
		'content': 'Lorem ipsum.'
	}
}
```

### `/`

This only renders the `home.jinja2` template. It passes to it a `blog` parameter so the Jinja template can use it to display the name of the blog and a list of posts:

```python
@app.route('/')
def home():
	return render_template('home.jinja2', blog=blog)
```

### `/post/<post_id>`

When a user wants to find a specific post, they will type in their browser something like `/post/2`. The `2` will then be assigned to the `post_id` parameter in the function.

This function then tries to find the correct blog post in our global variable.

If it is not found, it displays the `404.jinja2` page. If it is found, it displays the `post.jinja2` page:

```python
@app.route('/post/<int:post_id>')
def post(post_id):
	post = blog['posts'].get(post_id)
	if not post:
		return render_template('404.jinja2', message=f'A post with id {post_id} was not found.')
	return render_template('post.jinja2', post=blog['posts'].get(post_id))
```

### `/post/create`

This is the more complicated endpoint. It can be accessed either via `GET` or `POST` (what are HTTP verbs?).

If accessed via `GET`, we display the new post form.

When submitted, the form sends a request via `POST` which includes the form data. This endpoint accesses the data and creates a new post inside our global variable with that data.

```python
@app.route('/post/create', methods=['GET', 'POST'])
def create():
	if request.method == 'POST':
		title = request.form.get('title')
		content = request.form.get('content')
		post_id = len(blog['posts'])
		blog['posts'][post_id] = {'id': post_id, 'title': title, 'content': content}
		return redirect(url_for('post', post_id=post_id))
	return render_template('create.jinja2')
```