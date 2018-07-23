If you followed the tutorial [[Simple Password Authentication]], you will have a `UserSessionsController` with a `create` action very much like this one:

```ruby
def create
  if @user = login(params[:email], params[:password])
    redirect_back_or_to(:users, notice: 'Login successful')
  else
    flash.now[:alert] = 'Login failed'
    render action: 'new'
  end
end
```

Often this is sufficient. `login` will tell you if authentication was successful or not.

But sometimes you want to know *why* authentication failed. There are three cases to consider:

* The user provided a wrong login.
* The user provided a wrong password.
* The user is not actived yet.

You might want to display different error messages in the different cases <sup>[1](#footnote1)</sup> or otherwise react to one or more of the cases. All of this is possible when providing a block to `login`. `login` will call the block and feed it two parameters: `user` and `failure`. `user` works just like the return value of `login` if called without a block. It contains the user instance if authentication was successful and `nil` otherwise. If authentication failed, `failure` will contain a symbol you can use to find out why.

Putting this all together, your `create` action above could look like this instead:

```ruby
def create
  login(params[:email], params[:password]) do |user, failure|
    if user
      redirect_back_or_to(:users, notice: 'Login successful')
    else
      case failure
      when :invalid_login
        flash.now[:alert] = 'Wrong login provided.'
      when :invalid_password
        flash.now[:alert] = 'Wrong password provided.'
      when :inactive
        flash.now[:alert] = 'Your have not yet activated your account.'        
      end
      render action: 'new'
    end
  end
end
```

***

<b id="footnote1">[1]:</b> Please note that this is sometimes considered a security risk. Seperate error messages for wrong login and wrong password might be considered an information leak. It can be used to determine which username is already taken. In case of email addresses as login this means that one can easily find out if a certain person has made an account on the site in question. Then again, your signup form might already do this anyway.

***

***
