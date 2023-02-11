---
title: "Django Tutorial Part5 - Ideas For More Tests - Check for Choice"
date: 2023-02-10T12:23:17-06:00
draft: false
tags: ["python", "django"]
---

I'm going through the official [Django Tutorial](https://docs.djangoproject.com/en/4.1/intro/tutorial01/) and right now I'm at [Part 5](https://docs.djangoproject.com/en/4.1/intro/tutorial05/), doing Tests.
I tried to implement all the suggestions [further down](https://docs.djangoproject.com/en/4.1/intro/tutorial05/#ideas-for-more-tests):

- update the ResultsView to only show past Questions and exlude unpublished/future ones
- check for Questions without Choices and exclude them
- logged-in admin users should see unpublished Questions

So let's get going!

## Update the ResultsView to only show past Questions and exlude unpublished/future ones


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

## Check for Questions without Choices and exclude them

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

**Mission accomplished!**

## Logged-in admin users should see unpublished Questions

*I'm still working on that one, since I have to many ideas how to represent the unpublished questions for an admin*
