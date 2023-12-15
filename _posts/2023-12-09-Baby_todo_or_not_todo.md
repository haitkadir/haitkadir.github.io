---
title: Baby_todo_or_not_todo
date: 2023-12-14 16:53:16 +0800
categories: [Hackthebox-Challenges, Baby_todo_or_not_todo]
tags: [htb, ctf, pentesting, web, code-auditing, broken-access-control]
---


<img src="../../assets/global/banner.png" alt="banner image">

## Baby_todo_or_not_todo is a python flask chalenge on Hackthebox

### Description:

Baby todo is a simple todo list challenge that creates, mark as complate, and delete,\
for each `todos` on a per-user basis

### Source code:
<a href='https://mega.nz/file/OexhiY5B#Q_Zv1UmJpz6XH365b_XSArulT7LUWtLQWZ3etvwLu5w'>Download source code</a>

*zip password: `hackthebox`*

<img src="../../assets/baby-todo/home.png" alt="banner image">

### Code Auditing:


the two main important parts of the code is `routes`: 

```python
from flask import Blueprint, request, jsonify, session, render_template, g
from application.util import verify_integrity
from application.models import todo

main  = Blueprint('main', __name__)
api   = Blueprint('api', __name__)

@main.route('/')
def index():
	context = {
		'list_access': g.user,
		'secret': todo.get_secret_from(g.user)
	}
	return render_template('index.html', **context)

@api.before_request
@verify_integrity
def and_then(): pass

# TODO: There are not view arguments involved, I hope this doesn't break
# the authentication control on the verify_integrity() decorator
@api.route('/list/all/')
def list_all():
	return jsonify(todo.get_all())

@api.route('/list/<assignee>/')
def list_tasks(assignee):
	return jsonify(todo.get_by_user(assignee))

@api.route('/add/', methods=['POST'])
def add():
	todo.add(g.name, g.user)
	return {'success': f'Successfuly added {g.name} by user {g.user}'}

@api.route('/rename/<int:todo_id>/<new_name>/')
def rename_task(todo_id, new_name):
	g.selected.rename(new_name)
	return {'success': f'Successfuly edited {todo_id} to {new_name}'}

@api.route('/delete/<int:todo_id>/', methods=['DELETE'])
def delete(todo_id):
	g.selected.delete()
	return {'success': f'Successfuly deleted {todo_id}'}

@api.route('/complete/<int:todo_id>/')
def complete(todo_id):
	g.selected.complete()
	return {'success': f'Successfuly completed {todo_id}'}

@api.route('/assign/<int:todo_id>/<new_assignee>/')
def assign(todo_id, new_assignee):
	g.selected.reassign(new_assignee)
	return {'success': f'Successfuly reassigned {todo_id} to {new_assignee}'}
```
in the `routes` part we have some routes but the interesting one is `/list/all`\
if we can assecc this `route` we can list all todos stored in the `database`,\
even the admin ones which contains our `flag`

```python
INSERT INTO `todos` (`name`, `done`, `assignee`) VALUES
	('HTB{f4k3_fl4g_f0r_t3st1ng}', 0, 'admin');
```

if we intercept a request using `burpSuite`

<img src="../../assets/baby-todo/intercept1.png" alt="intercept1 image">

We can se a `cookie` is being sent with the request and it contains a user generated randomly.\
And alos a `secret`.

Now if let's try to access all todos by changing the endpoint from `/api/list/user<user>/?secret=<secret>` to `/api/list/all` endpoint and see what will happen.
<img src="../../assets/baby-todo/intercept2.png" alt="intercept2 image">

We got `not allowed`

let's check the `check_integrity()`:
to understand why we are not allowed.
check my comments below

```python
from flask import session, request, abort, g
import string, functools, random, re 
from application.models import todo

generate = lambda x: ''.join([random.choice(string.hexdigits) for _ in range(x)])

def verify_integrity(func):
	def check_secret(secret, name):
		if secret != todo.get_secret_from(name):
			return abort(403)

	@functools.wraps(func)
	def check_integrity(*args, **kwargs):
		g.secret = request.args.get('secret', '') or request.form.get('secret', '')
        # here it stores the secret if it exists

		if request.view_args: # checks if the view argumets is being passed if not it does nothing, vew argumets in flask is for example `/list/<this ones>/`
			list_access = request.view_args.get('assignee', '')

			if list_access and list_access != g.user:
				return abort(403)

			todo_id = request.view_args.get('todo_id', '')
			if todo_id:
				g.selected = todo.get_by_id(todo_id)

				if g.selected: 
					if dict(g.selected).get('assignee') == g.user:
						check_secret(g.secret, g.user)
						return func(*args, **kwargs)
					
					return abort(403)

				return abort(404)

		if request.is_json: # checks if any json data is passed if not it does nothing
			g.task = request.get_json()
			g.name = g.task.get('name', '')

			if g.name and len(g.name) <= 100 and not re.search('script|meta|link|src|on[a-z]', g.name, re.IGNORECASE):
				g.name = g.name.replace('<', '&lt;').replace('>', '&gt;')
				check_secret(g.task.get('secret', ''), g.user)
				return func(*args, **kwargs)

			return abort(400)
		
		check_secret(g.secret, g.user) # The important part of this function it checks the secret so if the secret is valid it will pass this is a `broken access control` vulnerability

		return func(*args, **kwargs)
	return check_integrity
```
`broken access control` vulnerability

In summary if no `view argumets` and no `json data` is passed,\
it will check only the `secret` if it's valid you'll get all todos stored in the database even the admin ones.





### Vulnerable code:
```python
def verify_integrity:
```

### Solve:

1 Intercept request.\
2 change the endpoint to `/api/list/all/?secret=<secret>` and don't forget the pass the secret and you'll get the flag.

<img src="../../assets/baby-todo/intercept3.png" alt="intercept3 image">






