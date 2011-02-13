### Controller

The core controller API is extremely short and simple:

login(*credentials) => authenticated user or nil if not found

logout

logged_in? => true or false

logged_in_user => authenticated user or false if not logged in


