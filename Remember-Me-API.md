### Controller

Use the good old login method with a third param.
This is usually the result of a "Remember me" checkbox from a form.
If it's anything that evals to true, it will remember.

@user = login(params[:email],params[:password],params[:remember])

To "forget me" just logout.

If you need finer control you can use the methods:

remember_me!

forget_me!