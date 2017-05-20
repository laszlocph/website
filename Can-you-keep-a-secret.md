

A typical question when you enter a room full of Docker enhusiasts, is how you handle secrets. Besides the other infamous one of how you handle persistence - spoiler you just don't.

The answer I have here might someone: security is not solved by a silver bullet, nor built in secret handling is the answer for everything. The truth is: the secret will be known at some point to someone, one can only limit its exposure for the intended user, for the short time of usage.

Let's take the options we have with Kubernetes.

## Put it in the yaml definitions
