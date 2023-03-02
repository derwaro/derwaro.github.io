---
title: "Django Tutorial Part5 - Ideas For More Tests - Check for Choice"
date: 2023-02-10T12:23:17-06:00
lastmod: :gitmod
draft: false
tags: ["python", "django"]
---

I'm going through the official [Django Tutorial](https://docs.djangoproject.com/en/4.1/intro/tutorial01/) and right now I'm at [Part 5](https://docs.djangoproject.com/en/4.1/intro/tutorial05/), doing Tests.
I tried to implement all the suggestions [further down](https://docs.djangoproject.com/en/4.1/intro/tutorial05/#ideas-for-more-tests):

- [update the ResultsView to only show past Questions and exlude unpublished/future ones](#first)
- [check for Questions without Choices and exclude them](#second)
- [logged-in admin users should see unpublished Questions](#third)

So let's get going!

## Update the ResultsView to only show past Questions and exlude unpublished/future ones {#first}


The first suggestion is merely a repetion of what was already covered in the tutorial for the `DetailView` adapted to the `ResultView`.



`polls/tests.py`:

```python
(...)
class ResultDetailViewTests(TestCase):
    def test_future_question(self):
        """
        the result page of a question with a future pub_date returns a 404 page
        """
        future_question = create_question(question_text="future q", days=5)
        url = reverse("polls:results", args=(future_question.id,))
        response = self.client.get(url)

        self.assertEqual(response.status_code, 404)

    def test_past_question(self):
        """
        the result page for a question with pub_date in the past shows the corresponding page
        """
        past_question = create_question(question_text="past q", days=-5)
        url = reverse("polls:results", args=(past_question.id,))
        response = self.client.get(url)

        self.assertContains(response, past_question.question_text)
```

in `polls/views.py`:

```python
(...)
class ResultsView(generic.DetailView):
    model = Question
    template_name = "polls/results.html"

    def get_queryset(self):
        return Question.objects.filter(
            pub_date__lte=timezone.now()
        )  # excludes future questions
```

**Done!**

## Check for Questions without Choices and exclude them {#second}

This one is a bit trickier, since we have to modify the `create_question` function, which was added in the tutorial:
```python
def create_question(question_text, days):
    """
    Create a question with the given `question_text` and published the
    given number of `days` offset to now (negative for questions published
    in the past, positive for questions that have yet to be published).
    """
    time = timezone.now() + datetime.timedelta(days=days)
    return Question.objects.create(question_text=question_text, pub_date=time)
```

this function will now include an optional `Choice`. Therefore we have to import this model and modify the function accordingly. In my first attempt I added the argument to the function with a default of `None`:

```python
#First attempt
(...) #existing imports

from .models import Question, Choice #added Choice model


def create_question(question_text, days, choice=None):
    """
    Create a question with the given `question_text` and published the
    given number of `days` offset to now (negative for questions published
    in the past, positive for questions that have yet to be published).
    """
    time = timezone.now() + datetime.timedelta(days=days)
    question = Question.objects.create(question_text=question_text, pub_date=time)

    if choice:
        Choice.objects.create(question=question, choice_text="this is a choice", votes=0)

    return question

(...)

```

Running the tests with this function, breaks various testcases:
```shell
(...)
FAIL: test_future_question_and_past_question (polls.tests.QuestionIndexViewTests)
(...)
FAIL: test_past_questions (polls.tests.QuestionIndexViewTests)
(...)
FAIL: test_two_past_questions (polls.tests.QuestionIndexViewTests)
(...)
AssertionError: Lists differ: [] != [<Question: Past question 2.>, <Question: Past question 1.>]

Second list contains 2 additional elements.
First extra element 0:
<Question: Past question 2.>

- []
+ [<Question: Past question 2.>, <Question: Past question 1.>]
(...)
```

whelp! Many minutes of reading my code passed until I found out why! I had already updated my `get_queryset` in `polls/views.py` to exclude all questions without assigned Choice:

```python
class IndexView(generic.ListView):
    template_name = "polls/index.html"
    context_object_name = "latest_question_list"
    

    def get_queryset(self):
        """
        Return the last five published questions (not including those set to be
        published in the future).
        """
        return (
            Question.objects.exclude(choice__isnull=True)
            .filter(pub_date__lte=timezone.now())
            .order_by("-pub_date")[:5]
        )
```

So since my new `create_question` function defaulted to not add a Choice it broke my tests!
Now to update and fix my `create_question`:



```python
def create_question(
    question_text, days, choice=True):  # choice must default to True, so that tests that don't focus on Choice existence succeed
    """
    Create a question with the given `question_text` and published the
    given number of `days` offset to now (negative for questions published
    in the past, positive for questions that have yet to be published).
    """
    time = timezone.now() + datetime.timedelta(days=days)
    question = Question.objects.create(question_text=question_text, pub_date=time)

    if choice == True:
        Choice.objects.create(
            question=question, choice_text="this is a choice", votes=0
        )

    return question
```

So to accomplish the test testing that Questions whithout Choices are not visible on the `IndexView` i came up with the following tests:

```python
(...)
class QuestionIndexViewTests(TestCase):
    (...)
    def test_question_with_no_choice(self):
        """Questions without Choices are not published"""
        question_without_choices = create_question(
            question_text="Q w/o Choice", days=-1, choice=False
        )
        response = self.client.get(reverse("polls:index"))
        self.assertQuerysetEqual(
            response.context["latest_question_list"],
            [],
        )

    def test_question_with_choice(self):
        """Q with Choices are published"""
        question_with_choices = create_question(
            question_text="Q w Choice", days=-1, choice=True
        )
        response = self.client.get(reverse("polls:index"))
        self.assertQuerysetEqual(
            response.context["latest_question_list"],
            [question_with_choices],
        )

    def test_question_with_and_without_choice(self):
        """only questions with choice are published. those without not."""
        question_without_choices = create_question(
            question_text="Q w/o Choice", days=-1, choice=False
        )
        question_with_choices = create_question(
            question_text="Q w/ Choice", days=-1, choice=True
        )

        response = self.client.get(reverse("polls:index"))
        self.assertQuerysetEqual(
            response.context["latest_question_list"],
            [question_with_choices],
        )
(...)
```

**Done!**

## Logged-in admin users should see unpublished Questions {#third}

the instructions are not completly clear about what "unpublished" questions are. I assumed it meant questions with a `pub_date` in the future (another variation would be that "unpublished" additionally includes Questions without Choices, as in the former suggested tests).

To make this happen, we first have to adapt our `IndexView` in `views.py`. I included a simple check for `superuser` or not. and based on this check create the corresponding `return` object.

```python
(...)

class IndexView(generic.ListView):
    template_name = "polls/index.html"
    context_object_name = "latest_question_list"

    def get_queryset(self):

        if (
            self.request.user.is_superuser # this checks if the currently logged in user is a superuser. Important to do it via `**self.**request` as only `request` can not be accessed in class based views.
        ):  # request in class based views is accessible via `self.request`` instead of `request`
            return Question.objects.exclude(choice__isnull=True).order_by("-pub_date")[
                :5
            ]  # returns published and unpublished questions for superusers

        else:  # returns only published Questions, as user is normal user or not logged in.
            return (
                Question.objects.exclude(choice__isnull=True)
                .filter(pub_date__lte=timezone.now())
                .order_by("-pub_date")[:5]
            )

(...)
```

Since this is mainly about writing tests, we also have to add some tests to our `tests.py`. In the style of `create_question()` i defined a `create_user()` function and used that function to create two separate tests for the `QuestionIndexViewTests` TestCases.

Right after the existing `create_question()` function I define the following function `create_user()`:

```python
(...)

def create_user(username, is_superuser=False, password=None):
    """create a user with given username and set superuser to True or False"""
    if is_superuser == False:
        user = User.objects.create_user(username)
    elif is_superuser == True:
        user = User.objects.create_superuser(username, password=password)
    user.save()

    return user

(...)
```

Take note of `password=None`: I initially created the function without this argument, which didn't work out, but also didn't give me an error message, which explained why the test failed. The reason is that a superuser absolutely must have a password set. So if we can't pass the `password` argument the user is **not** created and no error thrown at us! This obviously is in the Django docs:  `create_superuser()` must receive a `password` to work correctly.
Another thing that cost me too much time debugging: I forgot to `return` the `user` object created. Didn't work, just return it, please.

Summing up: We can now call this function with just a username and it will return us a normal user. If we set `is_superuser` to `True` **and** pass some `password` (e.g. "supersecurepassword") to it, it will return us dutifully a superuser!

Now, let's put it to work and test our `IndexView` with the following addiotional tests:

```python {hl_lines=[9]}
(...)
class QuestionIndexViewTests(TestCase):
    (...)

    def test_question_visibility_for_superuser(self):
        """unpublished questions are only visible to logged in superusers. For normal users unpublished questions will not be shown"""
        self.client = Client()
        self.user = create_user("testAdmin", is_superuser=True, password="password")
        self.client.force_login(self.user) 

        published_question = create_question(question_text="published Q", days=-1)

        unpublished_question = create_question(question_text="unpublished Q", days=2)

        response = self.client.get(reverse("polls:index"))
        self.assertQuerysetEqual(
            response.context["latest_question_list"],
            [unpublished_question, published_question],
        )

    def test_question_visibility_for_user(self):
        """unpublished questions are not visible to normal users."""
        create_user("testUser")
        published_question = create_question(question_text="published Q", days=-1)

        unpublished_question = create_question(question_text="unpublished Q", days=2)

        response = self.client.get(reverse("polls:index"))
        self.assertQuerysetEqual(
            response.context["latest_question_list"],
            [published_question],
        )

(...)
```

ad `self.client.force_login(self.user)`: Not sure if I absolutely necessarily need to force the login of the created superuser, but I did it just to be sure, after lots of debugging, which actually was caused by the missing `password` argument I described a bit up. 

**Done!**
