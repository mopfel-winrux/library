# Library

Simple library application. 

![Made with %graph-store](https://img.shields.io/badge/Made%20with-%25graph--store-darkblue)

## Features

Using this application, you will be able to:

- Create libraries, where a library is a collection of books
- Add and remove books from a library, if you are the owner
- Allow other's to view your library based on various permissioning schemes (policies)
- Add and remove comments from a library, if you are the owner or were granted access


## Installation

First, clone this repository in the directory that your piers are stored in.

```
$ git clone https://github.com/ynx0/library
```

Then, `|mount` the clay filesystem on each ship.

```
:dojo> |mount %
```

Then, run the install script. This copies the source code of the library app into the `%home` desk.

```
$ ./library/install.sh <path/to/the/pier>
```

Finally, `|commit` the `%home` desk in both ships

```
:dojo> |commit %home
```

You should see output similar to the following:

```
+ /~zod/kids/2/sur/library/hoon
+ /~zod/kids/2/mar/library/action/hoon
+ /~zod/kids/2/mar/library/command/hoon
+ /~zod/kids/2/app/library-proxy/hoon
+ /~zod/kids/2/lib/library/hoon
+ /~zod/kids/2/mar/library/response/hoon
+ /~zod/kids/2/mar/graph/validator/library/hoon
```

## Usage

*Note: The following section assumes you have multiple (fake) ships running. We'll user **~zod** and **~nus**, but feel free to use any ones you like.*

First, follow the installation instructions for two ships, ~zod and ~nus.

Then, go ahead and start the `%library-proxy` app on both ships.

```
:dojo> |start %library-proxy
```


To get baseline, run the following two commands:

This first one prints out the state of the graphs in graph store.
```
~zod:dojo> :graph-store +dbug [%state 'graphs']
>   {[p=[entity=~zod name=%dm-inbox] q=[p={} q=[~ %graph-validator-dm]]]}
```

This second one prints out the state of the library-proxy app.
```
~zod:dojo> :library-proxy +dbug
>   [%0 readers={} policies={}]
```

The output shows us that there is currently only one empty graph reserved for dms,
and that library proxy is currently not tracking any readers or policies.


Now, let's create a library on ~zod.

We'll create it with the `%open` policy, which means that anyone can request for access to the library.

**TODO** rename library to %sci-fi-collection

```
~zod:dojo> :library-proxy &library-command [%create-library [our %library1] [%open ~]]
```

Let's verify that the library exists.
```
~zod:dojo> :graph-store +dbug [%state 'graphs']
>   { [p=[entity=~zod name=%library1] q=[p={} q=[~ %graph-validator-library]]]
  [p=[entity=~zod name=%dm-inbox] q=[p={} q=[~ %graph-validator-dm]]]
}
```
We can see that there is now a new empty graph with the name we specified,
and that it is indeed using the validator for our library application.

Let's also verify that we've successfully recorded the policy for our library.
```
> ~zod:dojo> :library-proxy +dbug
>   [%0 readers={} policies={[p=[entity=~zod name=%library1] q=[%open ~]]}]
```

From this point on, the output will only contain important bits of information, 
skipping over what's not necessary.


Now, let's add a book to our library. Note that the isbn must be either length 10 or 13.
```
~zod:dojo> :library-proxy &library-command [%add-book [our %library1] ["Dune123" "0441172717"]]
```

Verify that the book was created successfully.
TODO collapsable summary/details
```
~zod:dojo> :graph-store +dbug [%state 'graphs']

{ [ p=[entity=~zod name=%library1]
      q
    [   p
      { [ key=170.141.184.505.110.303.839.596.375.394.968.666.112
            val
          [   post
            [ %.y
                p
              [ author=~zod
                index=~[170.141.184.505.110.303.839.596.375.394.968.666.112]
                time-sent=~2021.6.18..16.30.00..595d
                contents=~
                hash=~
                signatures={}
              ]
            ]
              children
            [ %graph
                p
              { [ key=8.319.395.793.566.789.475
                    val
                  [   post
                    [ %.y
                        p
                      [ author=~zod
                        index=~[170.141.184.505.110.303.839.596.375.394.968.666.112 8.319.395.793.566.789.475]
                        time-sent=~2021.6.18..16.30.00..595d
                        contents=~
                        hash=~
                        signatures={}
                      ]
                    ]
                    children=[%empty ~]
                  ]
                ]
                [ key=1.635.018.093
                    val
                  [   post
                    [ %.y
                        p
                      [ author=~zod
                        index=~[170.141.184.505.110.303.839.596.375.394.968.666.112 1.635.018.093]
                        time-sent=~2021.6.18..16.30.00..595d
                        contents=~
                        hash=~
                        signatures={}
                      ]
                    ]
                      children
                    [ %graph
                        p
                      { [ key=1
                            val
                          [   post
                            [ %.y
                                p
                              [ author=~zod
                                index=~[170.141.184.505.110.303.839.596.375.394.968.666.112 1.635.018.093 1]
                                time-sent=~2021.6.18..16.30.00..595d
                                contents=~[[%text text='Dune123'] [%text text='0441172717']]
                                hash=~
                                signatures={}
                              ]
                            ]
                            children=[%empty ~]
                          ]
                        ]
                      }
                    ]
                  ]
                ]
              }
            ]
          ]
        ]
      }
      q=[~ %graph-validator-library]
    ]
  ]
}
```

Although all the data is present and valid, it is not in a human readable format.
To understand the underlying structure, take a look at 
[this section(**TODO**)](code_architecture.md#schema) of the documentation
for an explanation of the schema.

<!--

Now, let's edit title of the book to remove the unnecessary numbers.

```
TODO this isn't implemented
```

Verify that a revision node was created successfully.
```
:graph-store +dbug [%state 'graphs']
```
-->


Now, let's bring ~nus into the picture.

First, get the list of available libraries from ~zod
```
~nus:dojo> :~zod/library-proxy &library-action [%get-libraries ~]
{[entity=~zod name=%library1]}
```
We see one library, with the name library1.

Let's request this one from ~zod.

```
~nus:dojo> :library-proxy &library-command [%request-library [~zod %library1]]
```

If we inspect **~nus**'s graph store, we will now see that it has fresh graph from **~zod** meant for library1.
```
~nus:dojo> :graph-store +dbug [%state 'graphs']
[p=[entity=~zod name=%library1] q=[p={} q=[~ %graph-validator-library]]]
```
However, there are no books yet that have been populated. 
In Library, we only keep store data and keep track of books that we are interested.

So, let's figure out what books are available on ~zod/library1.
```
~nus:dojo> :~zod/library-proxy &library-action [%get-books [~zod %library1]]
TODO output
```

Right now, we see the index of one book. Let's give it a face for easy access.

```
=top-of-dune 170.141.184.505.110.303.839.596.375.394.968.666.112
```

Now let's request this book.
```
~nus:dojo> :library-proxy &library-command [%request-book [~zod %library1] top-of-dune]
```

And verify that we got the update.
```
~nus:dojo> :graph-store +dbug [%state 'graphs']
TODO output
```


Take a look at ~zod's `%library-proxy` and notice how its state has updated information regarding who's tracking what resource:
```
~zod:dojo> :library-proxy +dbug
TODO output
```

As a result of the last two actions on ~nus's part, ~zod's `%library-proxy` now knows that 
(a) ~nus has requested and succesfully been granted access to %library1, and
(b) ~nus is interested in tracking updates to the book with index *`170.141.184.505.110.<...>`*, (which corresponds to Dune)


After having read the book, ~nus would like to comment about it.
```
~nus:dojo> :~zod/library-proxy &library-action [%add-comment [~zod %library1] top-of-dune "dune is ok"]
```

After thinking for a second, ~nus realizes she didn't complete his thought, so she writes another comment.
```
~nus:dojo> :~zod/library-proxy &library-action [%add-comment [~zod %library1] top-of-dune "dune is ok"]
```

Let's make sure ~nus's comment was properly sent to ~zod.
```
~zod:dojo> :graph-store +dbug [%state 'graphs']
TODO output
```

Now, ~nus feels like deleting her first comment, and does so:
```
~nus:dojo> :~zod/library-proxy &library-action [%remove-comment [~zod %library1] ~[top-of-dune %comments <index-frag of comment by ~nus>]]
```

~zod wants to clean up ~nus's second comment, because it doesn't really make sense without the first one.
So he ends up deleting it, since he's the owner of the library.

```
~zod:dojo> :library-proxy &library-action [%remove-comment [our %library1] ~[top-of-dune %comments <index-frag of other comment by ~nus>]]
```

Now, if we look at the graph store states for both ~zod and ~nus, we'll see the above changes reflected on both graphs.
The book nodes on both graphs are identical.
```
~zod:dojo> :graph-store +dbug
~nus:dojo> :graph-store +dbug
TODO output
```

Now, ~zod decides to delete the book *Dune*
```
:library-proxy &library-command [%remove-book [our %library1] top-of-dune]
```

Looking at the state,

```
~zod:dojo> :graph-store +dbug [%state 'graphs']
~nus:dojo> :graph-store +dbug [%state 'graphs']
TODO output
```

We can see that the book has been removed from both users' graph store.

We can also see that on ~zod's library proxy,
```
~zod:dojo> :library-proxy +dbug
TODO output
```

the index for the book is now removed from ~nus's tracked books on %library1


To wrap up, ~zod deletes the library.
```
~zod:dojo> :library-proxy &library-command [%remove-library [our %library1]]
```

Looking at the state,

```
~zod:dojo> :graph-store +dbug [%state 'graphs']
~nus:dojo> :graph-store +dbug [%state 'graphs']
TODO output
```

we can see that the library has been removed from all graph stores.

Also note that on ~zod's library proxy, %library1 is no longer tracked by ~nus.
```
~zod:dojo> :library-proxy +dbug
TODO output
```


## Advanced Usage

There are three different access policies supported by Library:

- `%open` - anyone can request the library
- `%children` - only children can request the library
- `%whitelist` - only select ships specified at the time of creation can request the library


<!--
## Reference

View the reference [here](https://zombo.com)
-->
