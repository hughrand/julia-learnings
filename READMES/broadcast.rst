2018-11-15 How arrays get expanded with broadcasting
====================================================

I was recently rewriting some code that needed to change with the v0.6 to v1.0 transition of julia. It was
interesting to be forced to think about one of the implications of broadcasting. The documentation available at
https://docs.julialang.org/en/v1/manual/arrays/#Broadcasting-1 indicates that "Julia provides broadcast, which
expands singleton dimensions in array arguments to match the corresponding dimension in the other array without using
extra memory, and applies the given function elementwise." It goes on to say that "Dotted operators such as .+ and .*
are equivalent to broadcast calls (except that they fuse, as described below)."

But what does this actually mean? Let me see if an example can help make this a little more concrete. We start with
a matrix that needs to have the columns scaled.
::
    #Set up a matrix
    X = [1 2 3 4 ; 1 2 3 2; 3 5 7 6; 5 3 5 6; 4 3 6 1]

    #Scale it in a clumsy way
    s       = sum(X,dims=1)              #the column sums
    S       = repeat(s,outer=size(X)[1]) 
    Xscaled = X ./ S

    #Scale it using broadcasting
    Xscaled = X ./ sum(X,dims=1)
    Xscaled = X ./ sum(X,dims=2)

After we think about it a bit, it becomes clear that Julia did exactly what it says it will do. It expanded the
1-D array produced by summing the columns and then divided element-wise. We will have to take the document's 
word that it didn't use extra memory.

Now, let's push our understanding by looking at row normalization.
::
    #Set up a matrix
    X = [1 2 3 4 ; 1 2 3 2; 3 5 7 6; 5 3 5 6; 4 3 6 1]

    #Scale it in a clumsy way
    sC       = sum(X,dims=1)              #the column sums
    sR       = sum(X,dims=2)              #the row    sums
    SC       = repeat(sC,outer=size(X)[1]) 
    SR       = reshape(repeat(sR,outer=size(X)[2]),size(X))  #This seems clumsy
    XCscaled = X ./ SC
    XRscaled = X ./ SR

    #Scale it using broadcasting
    XCBscaled = X ./ sum(X,dims=1)
    XRBscaled = X ./ sum(X,dims=2)

Interesting. The take-home here is that julia is clever enough to expand the 1-D array in a way that does just what
we want. For the column scaling, we get the row of scale factors concatenated together row-wise. For the row
scaling, we get the column of scale factors concatenated togehter column-wise. Very nice. I wonder if there is any
logic in julia for this, or if this just falls out of trying to treat all the matrix manipulations in a
consistent manner. But, that is for another time. For the moment we have two useful functions.
::
    "Column scale a 2D array."
    colscale(X::Matrix) = X ./ sum(X,dims=1)
    "Row scale a 2D array."
    rowscale(X::Matrix) = X ./ sum(X,dims=2)

Or, longer and a little less transparent, but maybe more suited to production code
::
   """
   Column scale a 2D array.

   # Examples
   ```jldoctest
   julia> X = [1 2 3; 1 2 3; 3 5 6; 5 3 5];

   julia> colscale(X)
   4×3 Array{Float64,2}:
   0.1  0.166667  0.176471
   0.1  0.166667  0.176471
   0.3  0.416667  0.352941
   0.5  0.25      0.294118
   ```
   """
   colscale(X::Matrix) = X ./ sum(X,dims=1)

   """
   Row scale a 2D array.

   # Examples
   ```jldoctest
   julia> X = [1 2 3; 1 2 3; 3 5 6; 5 3 5];

   julia> rowscale(X)
   4×3 Array{Float64,2}:
   0.166667  0.333333  0.5     
   0.166667  0.333333  0.5     
   0.214286  0.357143  0.428571
   0.384615  0.230769  0.384615
   ```
   """
   rowscale(X::Matrix) = X ./ sum(X,dims=2)

