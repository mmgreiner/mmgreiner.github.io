---
categories:
- programming
date: "2022-04-08T00:00:00Z"
tags:
- machine-learning
- F#
- dotnet
title: Machine Learning with ML.NET
showToc: true
---

Machine learning with [ML.NET] is documented in a confusing way, in particular when it comes to F#.

Some resources I found are:

- [ML.NET Cookbook](https://github.com/dotnet/machinelearning/blob/main/docs/code/MlNetCookBook.md#how-do-i-train-a-regression-model)
- [ML.NET Samples](https://github.com/dotnet/machinelearning-samples)
    Also has some F# samples, unfortunately not all.
- [Binary Classification for mushrooms](https://www.c-sharpcorner.com/article/how-to-do-binary-classification-in-asp-net-core-using-ml-net/) coded in C#, but interesting example.
- [FSharpML](https://csbiology.github.io/FSharpML//index.html) is a light weight API written in F# on top of [ML.NET].


## Supported classification

Machine learning falls into two big categories [[IBM]]: **supervised** and **unsupervised** learning.

*Supervised* learning algorithms are provided with a data set that has been prepared to contains the desired result, the *label*. *Unsupervised* learning does not have labels, but discovers patterns that help solve for clustering or association problems. 

Supervised learning covers two kinds of learning [[IBM]]: 

- **Classification** algorithms classify the data into specific categories.

- **Regression** algorithms are used to understand the relationship between dependent and independent variables. It is typically used for projections.

[mlnet] supports several of these algorithms, as indicated in the [tutorial]:

> - **Classification** - Use this when you want to predict which category data belongs in (for example, analyzing sentiment of customer reviews as either positive or negative).
>
> - **Image classification** - Use this when you want to predict which category an image belongs to (for example, predicting if an image is of a cat or a dog).
>
> - **Regression** (for example, value prediction) - Use this when you want to predict a numeric value (for example, predicting house price).
>
> - **Forecasting** - Use this when you want to forecast future values in a time-series (for example, forecast quarterly sales).
>
> - **Recommendation** - Use this when you want to recommend items to users based on historical ratings (for example, product recommendation).

Classifications can be either binary (true / false) or by category (cold, warm, hot).


## Typical steps


There are some typical steps to take during machine learning:

1. Data loading and preprocessing

1. Data processing / pipeline configuration and feature extraction

1. Train on data

1. Evaluate the model

1. Predict a new value



## Data loading

[ML.NET] doesn't work with record types, so all data columns are only referenced by their names. This can be confusing.

I'm using three data structues:

- one to read in the data
- one how it looks after featurization into vectors
- one for the predictions.

ta a simple example: learning a person's gender from weight and height.

The csv training file looks like this:

    "Gender","Height","Weight"
    "Male",73.847017017515,241.893563180437
    "Male",68.7819040458903,162.310472521300
    "Male",74.1101053917849,212.7408555565
    ...
    "Female",58.9107320370127,102.088326367840
    "Female",65.2300125077128,141.305822601420
    "Female",63.3690037584139,131.041402692995

### Data structure

The data structure I use:

~~~~fsharp
type Person = {
    [<ColumnName("Label")>]
    Gender: boolean
    Height: single
    Weight: single
}
~~~~

Couple things to notice:
- The label to be learned should be called `Label`. If not, it may cause all kinds of confusion. You can rename it using `ColumnName`.
- For binary classifiers, the label should be boolean
- Numeric data should be either `int` or `float32` or `single`. `float`will not compile.

Typically, I first use [FSharp.Data][fsharp-data] to read in the data. It also makes it very easy to read data from the web; and typically, the csv files are stored somewhere on github.

~~~~fsharp
open FSharp.Data
let csv = FSharp.Data.CsvFile.Load(githubrawUrl)
printfn "headers: %A" csv.Headers

let data' = csv.Rows |> Seq.map (fun r -> {
    Gender = r.["Gender"] = "Female"
    Height = single r.["Height"]
    Weight = single r.["Weight"]
})
~~~~

## Data preparation
Now we enter [mlnet]. It uses `DataView`s to handle all data. As input we can give a sequence of data.

~~~~~fsharp
open Microsoft.ML
open Microsoft.ML.Data
let mlcontext = MLContext(seed = Nullable 1)
let data = 
    mlcontext.Data.LoadFromEnumerable(data')
    |> mlcontext.Data.ShuffleRows

let dataSplit = mlcontext.Data.TrainTestSplit(data, testFraction = 0.3)
~~~~~

We are randomly shuffling the rows in order not to train on a pre-ordered dataset. 30% of the data is used for testing, 70% for training. Obviously, this split depends on the size of your input data.

## Feature extraction

This is done in [ML.NET] using `Transformation`s. 

~~~fsharp
open Microsoft.ML.Transforms

let dataPipeline =
    EstimatorChain()
        .Append(mlx.Transforms.NormalizeMinMax(outputColumnName = "fHeight", inputColumnName = "Height"))
        .Append(mlx.Transforms.NormalizeMinMax("fWeight", "Weight"))
        .Append(mlx.Transforms.Concatenate(nameof(PersonVector_.Features), "fWeight", "fHeight"))
~~~

The EstimatorChain, or pipeline steps, are build up of the following functions:

- *normalize* the given data for `Height` and `Weight` into values between 0.0 and 1.0. The newly created columns are calle `fHeight` and `fWeight`, indicating that they have been featurized.

- Now we create one array of all the so-called *features* by concatenating these columns into one column `"Features"`. 

### Inspect intermediate data

we would like to see whether the featurization process worked correctly. This is best done by defining a type `PersonVector`; it's fields correspond exactly to the output column names above.

Data rows are lazy loaded - this means, they are not actually processed until we fit the data.

The data view can be enumerated using `CreateEnumerable`.

~~~fsharp
[<CLIMutable>]
type PersonVector = {
    Label: bool
    Features: single[]
}

let transformedData = dataPipeline.Fit(data).Transform(data)
let d0 = mlx.Data.CreateEnumerable<PersonVector>(transformedData, reuseRowObject = false) 
printfn "first is %A" (Seq.head d0)
~~~~


## Train

We use a stochastic dual coordinate ascent (SDCA) method, as described [here](https://docs.microsoft.com/en-us/dotnet/api/microsoft.ml.trainers.sdcalogisticregressionbinarytrainer?view=ml-dotnet).

We also use a little trick to ensure that the column names correspond to the fields of the records.

~~~fsharp
// to get the names of the fields
let PersonVector_ = Unchecked.defaultof<PersonVector>

let trainer =
    mlx.BinaryClassification.Trainers.SdcaLogisticRegression(
        labelColumnName = nameof(PersonVector_.Label), 
        featureColumnName = nameof(PersonVector_.Features)
    )
let trainingPipeline = dataPipeline.Append(trainer)

// train the model
printfn "------ training"
let trainedModel = trainingPipeline.Fit(dataSplit.TrainSet)
~~~


[mlnet]:    https://dotnet.microsoft.com/en-us/learn/ml-dotnet
[fsharp-data]:  https://fsprojects.github.io/FSharp.Data/
[tutorial]:  https://dotnet.microsoft.com/en-us/learn/ml-dotnet/get-started-tutorial/create
[ML.NET]:   https://dotnet.microsoft.com/en-us/learn/ml-dotnet
[IBM]:      https://www.ibm.com/cloud/learn/supervised-learning#toc-how-superv-A-QjXQz-

[Accessible-AI]: https://accessibleai.dev/post/mlnet_fsharp_regression/
