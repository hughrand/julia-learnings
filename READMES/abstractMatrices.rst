2018-11-15 AbstractMatrices and the move to Julia 1.0
=====================================================

With Julia 1.0, it is important to make sure that functions that expect to ingest Arrays can ingest
AbstractArrays (Some info in https://github.com/JuliaArrays/StaticArrays.jl/issues/410). Otherwise
you can get some very unexpected behaviour. For example:
::
    add1(A::Matrix)         = A .+ 1
    add2(A::AbstractMatrix) = A .+ 2
    X = [1 2 3; 4 5 6; 7 8 9]
    add1(X)    #X is a Matrix,    so it works fine
    add2(X)    #X is a Matrix,    so it works fine
    add1(X')   #X' is a Matrix,   so it breaks
    add2(X')   #X' is an Adjoint, so it works fine

You can see the relationships by the following commands
::
    subtypes(AbstractMatrix)  #Adjoint is a subtype
    superType(Matrix)         #This leads to DenseArray
    supertype(DenseArray)     #This leads to AbstractArray


