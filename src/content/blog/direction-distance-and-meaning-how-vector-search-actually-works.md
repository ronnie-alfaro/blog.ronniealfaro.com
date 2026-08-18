---
title: "Direction, Distance, and Meaning: How Vector Search Actually Works"
description: "An intuitive guide to cosine similarity, Euclidean distance, and dot product—and the different questions each metric asks of an embedding space."
pubDate: 2026-08-17
---

One of the things I find slightly unfortunate about vector databases is that the terminology makes the whole subject sound considerably more mysterious than it really is. We talk about embeddings, cosine similarity, Euclidean distance, dot products and multidimensional spaces, and before long it feels as if we are doing advanced mathematics just to find a paragraph about cats. Underneath all of that, however, the problem is surprisingly intuitive: we have converted information into vectors, and now we need some way of deciding which vectors are related to the vector produced by our search.

The interesting part is that related can mean different things mathematically. Sometimes what matters is how close two points are. Sometimes what matters is whether they point in approximately the same direction. And sometimes both direction and magnitude matter. This is why vector databases support different similarity metrics, and why choosing between cosine similarity, Euclidean distance and dot product is more than a small implementation detail.

I normally explain embeddings using a city.

Imagine an enormous city containing every concept the embedding model has learned to represent. Somewhere in that city there is an area associated with animals. Inside that area there might be something resembling a neighborhood of mammals, and somewhere inside that neighborhood we eventually reach a group of buildings associated with cats. One building might contain general information about cats, another about veterinary care, another about breeds, and as we keep moving deeper into this imaginary structure we could eventually think of floors, rooms and even individual inhabitants as increasingly specific pieces of information.

Obviously an embedding space is not literally organized this way. There is no dimension called CATNESS, another called WHISKERS, and a conveniently placed elevator leading to SIAMESE. A real embedding might contain hundreds or thousands of dimensions, each contributing a tiny amount to the final position of a concept. But the city analogy gives us something our three-dimensional primate brains can actually work with.

If we reduce the problem to two dimensions, we can draw something like this:

```text
                           cats
                            *
                         *
                      *
                   *
                *
             O
```

The point O represents the origin, and the direction toward those points represents, in our deliberately simplified universe, the general direction of cat-related concepts.

Now imagine that I search for:

> Why do cats purr?

The embedding model converts that sentence into another vector. If the model has done its job reasonably well, that vector should point into approximately the same semantic region as vectors representing things such as cat, feline, purring, kitten or perhaps domestic animal behavior.

And this is where an important distinction appears. We can ask how close the vectors are, or we can ask whether they are pointing in the same direction.

Those are not the same question.

## Euclidean distance: how far away is the building?

Euclidean distance is probably the easiest of the three metrics to visualize because it is essentially the ordinary geometric distance between two points. In two dimensions it is the same formula most of us encountered in school:

$$
d(A,B)=\sqrt{(x_2-x_1)^2+(y_2-y_1)^2}
$$

In an embedding with $n$ dimensions, the same idea becomes:

$$
d(A,B)=\sqrt{\sum_{i=1}^{n}(A_i-B_i)^2}
$$

Nothing particularly exotic is happening here. If two vectors occupy very similar positions in the space, their Euclidean distance is small; if they are far apart, the distance is large.

Returning to the city analogy, Euclidean distance is basically asking:

> How many blocks away is that building?

If my query lands close to the building containing cat-related information, Euclidean distance considers it a good match.

But semantic search introduces a curious situation. Imagine two vectors that lie on exactly the same line from the origin:

```text
O -------- A ------------------------------ B
```

Suppose:

$$
A=(1,1)
$$

and:

$$
B=(10,10)
$$

These two vectors point in exactly the same direction, but they are far apart. Their Euclidean distance is:

$$
d(A,B)=\sqrt{(10-1)^2+(10-1)^2}
$$

which gives approximately:

$$
12.73
$$

So from the point of view of Euclidean distance, they are not particularly similar.

Directionally, however, they are identical.

And that is the idea that makes cosine similarity so useful.

## Cosine similarity: are we heading toward the same neighborhood?

Cosine similarity is much less interested in where the vectors end and much more interested in the angle between them.

Mathematically it is written as:

$$
\cos(\theta)=\frac{A\cdot B}{\lVert A\rVert\lVert B\rVert}
$$

The formula looks more intimidating than the concept. What we are really asking is simply: what is the angle between these two vectors?

```text
                         B
                       /
                     /
                   /
                 / θ
               /
O ------------→ A
```

If both vectors point in exactly the same direction, the angle between them is $0^\circ$, and:

$$
\cos(0^\circ)=1
$$

A cosine similarity of 1 therefore means that the vectors are perfectly aligned.

If they are perpendicular, the angle is $90^\circ$:

$$
\cos(90^\circ)=0
$$

And if they point in opposite directions:

$$
\cos(180^\circ)=-1
$$

The important part is that cosine similarity normalizes the magnitude of the vectors. A short vector pointing northeast and a vector ten times longer pointing northeast still have the same direction.

Using our city again, imagine I am standing somewhere near the center and I know that the cat neighborhood is northeast. One person tells me:

> Walk two blocks northeast.

Another tells me:

> Walk twenty blocks northeast.

Those instructions lead to very different locations, but they agree completely about the direction.

For some kinds of semantic search, that agreement is exactly what I care about. If my query is clearly pointing toward the conceptual region associated with cats, I may care much more about the fact that the stored vector points toward the same region than about the precise difference in their lengths.

This is what people mean when they say cosine similarity compares direction rather than distance.

It is not that distance somehow disappears from mathematics. It is that we deliberately normalize the vectors so their lengths stop dominating the comparison.

## Dot product: direction, but magnitude still matters

The dot product is closely related to cosine similarity, which becomes obvious when we write both formulas next to each other.

The dot product is:

$$
A\cdot B=\sum_{i=1}^{n}A_iB_i
$$

But geometrically it can also be written as:

$$
A\cdot B=\lVert A\rVert\lVert B\rVert\cos(\theta)
$$

Compare that with cosine similarity:

$$
\cos(\theta)=\frac{A\cdot B}{\lVert A\rVert\lVert B\rVert}
$$

Cosine similarity takes the dot product and divides away the magnitudes of the two vectors.

That is really the essential difference.

The dot product therefore cares about two things at the same time: how well the vectors align and how large they are.

Take a query vector:

$$
Q=(1,1)
$$

and two candidates:

$$
A=(2,2)
$$

$$
B=(10,10)
$$

Both point in exactly the same direction as $Q$, so cosine similarity gives both of them:

$$
1
$$

From the perspective of direction, they are equally good.

The dot products are different:

$$
Q\cdot A=(1)(2)+(1)(2)=4
$$

while:

$$
Q\cdot B=(1)(10)+(1)(10)=20
$$

So dot product strongly favors $B$.

Euclidean distance does almost the opposite. The distance from $Q$ to $A$ is about:

$$
1.41
$$

while the distance from $Q$ to $B$ is about:

$$
12.73
$$

Euclidean distance therefore strongly favors $A$.

And I think this single example explains the three metrics better than several pages of definitions.

```text
                    B
                   /
                  /
                 /
                A
               /
              /
             Q
            /
           O
```

Cosine similarity looks at those vectors and says:

> They are all pointing in essentially the same direction.

Dot product says:

> They are pointing in the same direction, but B has a much stronger magnitude.

Euclidean distance says:

> A is physically much closer to Q.

All three answers are mathematically correct. They are simply answering different questions.

That distinction matters enormously when we use embeddings for retrieval, because there is no universal mathematical definition of “most relevant.” We first have to decide what kind of relationship our embedding model has encoded and what aspect of that relationship we want the search metric to preserve.

And this brings us back to the city.

If I am looking for information about cats, sometimes the most important thing is finding the nearest building. In that case, distance is a perfectly reasonable idea.

But sometimes I care less about whether I am standing three blocks or thirty blocks away. What matters is that I am walking toward the cat district rather than toward databases, French cuisine or medieval theology.

In those situations, direction tells me more than distance.

And that, stripped of most of the intimidating terminology, is the intuition behind cosine similarity.
