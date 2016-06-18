title: Various solutions for solving the Elm Guide age exercise
tags:
  - elm
  - elm-lang
categories:
  - elm
date: 2016-06-18 22:06:53
---


Step-by-step walkthrough of various solutions to the
[Elm Guide age challenge](http://guide.elm-lang.org/architecture/user_input/forms.html)
intended to help novice Elm users grasp types, type conversions and refactoring logic.

## Before We Begin

Please note that:

- all provided code can be tested online by copy-pasting into the [online Elm Try editor](http://elm-lang.org/try)
- this tutorial is aimed at new users with little to no prior knowledge of the [Elm](http://elm-lang.org/) 
functional programming language
- we assume users have read all information in the Guide up to and including the Forms page
- we will follow the natural learning curve of novice users in four steps (from most simple to more complex)

## The challenge

The Elm Guide contains various cleverly placed "exercises" challenging the user
to dig in and solve seemingly simple problems. The one we are addressing is:

> Add an additional field for age and check that it is a number.

We are picking up where the code example in the Guide stops so make sure to
at least understand that code or, even better, drop
[a copy](https://github.com/bravo-kernel/application-examples/blob/master/blog-various-solutions-to-the-elm-guide-age-excercise/starting-source.elm)
into the [online Elm Try editor](http://elm-lang.org/try).

## Solution 1.

Mimicing earlier instructions in the Guide this is probably the first solution
users will come up with:

- adding a new record field `age` with type `Int` to the Model
- making sure the new model field has a default value (`0`)
- extending update `Msg` with `Age String` (not Int because onInput always returns a string)
- extending update Case-Of with a new pattern for `Age`
- adding a new input field to the view
- adding a new if statement to the view `viewValidation` (with smart user feedback message)

This will either:
- update the record's `age` field with the given input value if a valid number was provided
- update the record's `age` field with hardcoded `Result.withDefault` value if `String.toInt` conversion fails


```haskell
import Html exposing (..)
import Html.App as Html
import Html.Attributes exposing (..)
import Html.Events exposing (onInput)
import String

main =
  Html.beginnerProgram
    { model = model
    , view = view
    , update = update
    }

-- MODEL

type alias Model =
  { name : String
  , password : String
  , passwordAgain : String
  , age: Int
  }

model : Model
model =
  Model "" "" "" 0

-- UPDATE

type Msg
    = Name String
    | Password String
    | PasswordAgain String
    | Age String

update : Msg -> Model -> Model
update action model =
  case action of
    Name name ->
      { model | name = name }

    Password password ->
      { model | password = password }

    PasswordAgain password ->
      { model | passwordAgain = password }

    -- =====================================================
    -- Age get's passed in here as string by `onInput` so we
    -- need to convert it to Int as this is the type required
    -- by our Model definition. Since `String.toInt` conversion
    -- could fail (and thus break our application) we make
    -- sure to set default value to 0 using Result.
    -- =====================================================
    Age age ->
      { model | age = String.toInt age |> Result.withDefault 0}

-- VIEW


view : Model -> Html Msg
view model =
  div []
    [ input [ type' "text", placeholder "Name", onInput Name ] []

    -- =====================================================
    -- Please note that `onInput` will always produce a
    -- string so even when we change the type "string" below
    -- to "number" (so the browser will check for numbers)
    -- we still need to do the String to Int conversion
    -- described above.
    -- =====================================================
    , input [ type' "string", placeholder "Age", onInput Age ] []

    , input [ type' "password", placeholder "Password", onInput Password ] []
    , input [ type' "password", placeholder "Re-enter Password", onInput PasswordAgain ] []
    , viewValidation model
    ]

viewValidation : Model -> Html msg
viewValidation model =
  let
    (color, message) =
      if model.age < 18 then
        ("red", "You are not old enough to watch nude models yet")
      else if model.password == model.passwordAgain then
        ("green", "OK")
      else
        ("red", "Passwords do not match!")
  in
    div [ style [("color", color)] ] [ text message ]
```

Copy-paste this code into the online editor, fill in a name and you will see
the `age` field behaving as expected. Not bad for a first time Elm user, get
some coffee.

## Solution 2

By now we realize that solution 1 is far from DRY. If we would change the
model's default value we would have to update our code in two places (
the model's default value and `Result.withDefault` value).

The logical step to refactor would be using the model's default value so we
update our page a little, this time:

- replacing the existing Case-Of with in `update` a new case for `Age`
- returning the current record as-is (with it's default value for `age`) if String.toInt conversion fails
- updating the record's age field with the integer provided by the user if the conversion
succeeds

```haskell
import Html exposing (..)
import Html.App as Html
import Html.Attributes exposing (..)
import Html.Events exposing (onInput)
import String

main =
  Html.beginnerProgram
    { model = model
    , view = view
    , update = update
    }

-- MODEL

type alias Model =
  { name : String
  , password : String
  , passwordAgain : String
  , age: Int
  }

model : Model
model =
  Model "" "" "" 0

-- UPDATE

type Msg
    = Name String
    | Password String
    | PasswordAgain String
    | Age String

update : Msg -> Model -> Model
update action model =
  case action of
    Name name ->
      { model | name = name }

    Password password ->
      { model | password = password }

    PasswordAgain password ->
      { model | passwordAgain = password }

    -- =====================================================
    -- We still need to convert Age being passed in as type
    -- `String` to `Int` but this time we are not using 
    -- Result.withDefault to set the default value to `0`.
    -- Instead, if String.toInt conversion fails we simply
    -- return the record as-is (effectively using the model's
    -- default value).
    -- =====================================================
    Age age ->
      case String.toInt age of
        Result.Err err -> -- `err` will contain the error message, we could  use `_` and it would be ignored
          model
        Result.Ok age ->
          { model | age = age }

-- VIEW

view : Model -> Html Msg
view model =
  div []
    [ input [ type' "text", placeholder "Name", onInput Name ] []
    , input [ type' "string", placeholder "Age", onInput Age ] []
    , input [ type' "password", placeholder "Password", onInput Password ] []
    , input [ type' "password", placeholder "Re-enter Password", onInput PasswordAgain ] []
    , viewValidation model
    ]

viewValidation : Model -> Html msg
viewValidation model =
  let
    (color, message) =
      if model.age < 18 then
        ("red", "You're not old enough to watch nude models yet")
      else if model.password == model.passwordAgain then
        ("green", "OK")
      else
        ("red", "Passwords do not match!")
  in
    div [ style [("color", color)] ] [ text message ]
```

Again, copy-paste this code into the online editor. It should behave as expected. 

## Solution 3

Even though solution 2 is already a lot nicer we would still need to duplicate
a lot of Case-Of logic in the `update` layer if we were to add another input
field requiring an integer.

The logical step to refactor would be creating a custom function for the
type conversion so we can re-use the logic. Again we update our page,
this time:

- adding a custom function `inputStringToInt` (using `String` as parameter, returning `Int`)
- making the function return an Int if type conversion succeeds
- making the function return 0 if the type conversion fails

```haskell
import Debug
import Html exposing (..)
import Html.App as Html
import Html.Attributes exposing (..)
import Html.Events exposing (onInput)
import String

main =
  Html.beginnerProgram
    { model = model
    , view = view
    , update = update
    }

-- MODEL

type alias Model =
  { name : String
  , password : String
  , passwordAgain : String
  , age: Int
  }

model : Model
model =
  Model "" "" "" 0

-- UPDATE

type Msg
    = Name String
    | Password String
    | PasswordAgain String
    | Age String


update : Msg -> Model -> Model
update action model =
  case action of
    Name name ->
      { model | name = name }

    Password password ->
      { model | password = password }

    PasswordAgain password ->
      { model | passwordAgain = password }

    -- =====================================================
    -- Convert age to Int using custom function so we can
    -- re-use logic for other input fields requiring Ints.
    -- =====================================================
    Age age ->
      { model | age = inputStringToInt age }

-- VIEW

view : Model -> Html Msg
view model =
  div []
    [ input [ type' "text", placeholder "Name", onInput Name ] []
    , input [ type' "string", placeholder "Age", onInput Age ] []
    , input [ type' "password", placeholder "Password", onInput Password ] []
    , input [ type' "password", placeholder "Re-enter Password", onInput PasswordAgain ] []
    , viewValidation model
    ]

viewValidation : Model -> Html msg
viewValidation model =
  let
    (color, message) =
      if model.age < 18 then
        ("red", "You're not old enough to watch nude models yet")
      else if model.password == model.passwordAgain then
        ("green", "OK")
      else
        ("red", "Passwords do not match!")
  in
    div [ style [("color", color)] ] [ text message ]

-- CUSTOM FUNCTIONS

inputStringToInt : String -> Int

inputStringToInt input =
  let result = String.toInt input
  in case result of 
    Err msg -> 0 -- `msg` will contain the error message, we could use `_` instead and it would be ignored
    Ok converted ->  converted
```

By know you should know the drill so... copy, paste, enjoy and get some coffee
(the good part is coming next).

## Solution 4.

By now the concept of types should have landed and if things went well
there are probably two things on your mind right now:

1. the major flaw in all previous solutions: how to make `age` optional (since we don't want `0` as default value)
2. how on earth that would ever be possible knowing that Elm doesn't support `null` or `undefined`?

In all honesty, this is also probably the moment you should join the 
[Elm Slack](http://elmlang.herokuapp.com/) beginners channel and hope some of
the friendly people out there will help you out.

Luckily... we've  already done that and the solution for our problem is found
in the Elm core library
[Maybe](http://package.elm-lang.org/packages/elm-lang/core/4.0.1/Maybe), literally stating:

> A Maybe can help you with optional arguments, error handling, and **records with optional fields**.

For the last time we will update our page, this time:

- changing the model's `age` field type to `Maybe Int`
- changing the model's `age` field default value to `Nothing` (think of it as `null`)
- updating our custom function `inputStringToInt` to:
    - return `Maybe Int` (instead of `Int`)
    - return `Nothing` if the type conversion fails
    - return a `Just Int` value if the conversion succeeds

Please note that we also MUST replace the if statement inside
`viewValidation` with a Case-Of because standard Elm comparisons will only 
accept basic types like integers, strings, etc.

```haskell
import Html exposing (..)
import Html.App as Html
import Html.Attributes exposing (..)
import Html.Events exposing (onInput)
import String

main =
  Html.beginnerProgram
    { model = model
    , view = view
    , update = update
    }

-- MODEL

type alias Model =
  { name : String
  , password : String
  , passwordAgain : String
  , age: Maybe Int
  }

model : Model
model =
  Model "" "" "" Nothing

-- UPDATE

type Msg
    = Name String
    | Password String
    | PasswordAgain String
    | Age String -- we keep this a String because that's what we get from `onInput` (converted to Maybe Int before inserting into model)


update : Msg -> Model -> Model
update action model =
  case action of
    Name name ->
      { model | name = name }

    Password password ->
      { model | password = password }

    PasswordAgain password ->
      { model | passwordAgain = password }

    Age age ->
       { model | age = inputStringToInt age }

-- VIEW


view : Model -> Html Msg
view model =
  div []
    [ input [ type' "text", placeholder "Name", onInput Name ] []
    , input [ type' "string", placeholder "Age", onInput Age ] []
    , input [ type' "password", placeholder "Password", onInput Password ] []
    , input [ type' "password", placeholder "Re-enter Password", onInput PasswordAgain ] []
    , viewValidation model
    ]

viewValidation : Model -> Html msg

viewValidation model =
  let
    (color, message) =
      case model.age of
        Nothing ->
          ("red", "Please provide your age")
        Just age -> if age < 18 then
          ("red", "You're not old enough to watch nude models yet!")
          else if model.password == "" then
            ("red", "Please provide a password")                
          else if model.password == model.passwordAgain then
           ("green", "OK")
          else
            ("red", "Passwords do not match!")

  in
    div [ style [("color", color)] ] [ text message ]

-- CUSTOM FUNCTIONS

inputStringToInt : String -> Maybe Int

inputStringToInt input =
  let result = String.toInt input
  in case result of
    Err _ -> Nothing
    Ok converted -> Just converted
```

Copy, paste and be merry. We have not only completed the `age` challenge, in 
the process we learned about types and how to convert them while keeping
our code DRY and highly maintainable. Thank you Elm!

## Closing Notes

As with all learning curves these are obviously only the first baby steps into
the world called Elm but it should provide you with enough confidence to continue
with the next chapter of the Guide.

Another thing to note is that "there is more than one way to do it" and thus
the provided solutions are non-exhaustive and most likely a lot of different
approaches are possible and they may all be valid and or even better.

For example, the following feedback was provided on Slack:

```haskell
if condition then x else y
```
is equivalent to

```
case condition of
  True -> x
  False -> y
```

Which is more readable, and abstracts away the pattern
matching. I would always prefer using abstraction instead of direct pattern
matching in general, because that means if the implementations of the type
changes in the future (unlikely for Bool types) you won’t have to update every
single function that consumes the type. It’s effectively a way to achieve
encapsulation.
 
So still lots to explore, learn and try but remember... if it compiles it's good ;)
 
*Big thank you to Elm Community members 
[mattsenior](https://github.com/mattsenior), 
[szabba](https://github.com/szabba) and
[iteloo](https://github.com/iteloo)
for their help while creating this post.*
