#golang #context #software_engineering 

### Refrences:
https://www.youtube.com/watch?v=mfgBhGu5pco&t=1058s&ab_channel=GopherConUK


- Context is not a map it's a tree with each context having a "key" and "value" pair with a pointer to it's parent context.
- Upon querying for a key in a context, we check to see if the key matches the key of the current context (the child one we are working with) and if we were unable to find that key (and the value for that key) we start traversing the tree  to the root (emptyCtx) to check if we can find that key

# TODO