---
title: "State vs. Behaviour Verification"
date: 2020-11-22T10:15:00+02:00
tags: ["TDD", "Outside-In", "Acceptance Test", "Contract Tests"]
draft: true
---
{{< imgpreview src="/images/state_vs_behaviour/state_vs_behaviour.png">}}

Suppose we have the following interface:

{{< highlight kotlin >}}
interface Validator {
  fun validate(query: String): Boolean
}
{{< /highlight >}}

and suppose it's being used as a collaborator in a few places:

{{< highlight kotlin >}}
class FindItems(
  val validator: Validator,
  val repository: Repository
) {

  fun search(keyword: String) {
    if (validator.validate(keyword)) {
      repository.findItemsContaining(keyword)
    }
  }
}

class DeleteItems(
  val validator: Validator,
  val repository: Repository
) {

  fun deleteMatches(value: String) {
    if (validator.validate(value)) {
      repository.deleteItemsMatching(value)
    }
  }
}
{{< /highlight >}}

Of course, we would have some tests for these functionalities, which might look something like this respectively:

{{< highlight kotlin >}}
@Test
fun findMatches() {
  val validator = mock<Validator>()
  val lookup = FindItems(validator)
  given(validator.validate(query)).willReturn(true)

  lookup.search(query)

  validateTheOutputOfTheOperation
}

@Test
fun deletesMatches() {
  val validator = mock<Validator>()
  val cleanup = DeleteItems(validator)
  given(validator.validate(query)).willReturn(true)

  cleanup.deleteMatches(query)

  validateTheOutputOfTheOperation
}
{{< /highlight >}}

At this case, we are using a `mock` validator and we are setting it up to return specific result, in this case `true`.

Let's suppose that the initial implementation of the validator for the shipping code looks like this:

{{< highlight kotlin >}}
class DefaultValidator: Validator {
  override fun validate(query: String): Boolean {
    return query.trim().lenght > 3
  }
}
{{< /highlight >}}

Now, here are a bunch of interesting questions:
1. What's the relation between the shipping implementation of the validator, and the test-doubles?
2. How many tests would fail when a business rule for a valid query changes (say from `lenght > 3` to `length > 8`)?
3. How many tests should fail?
4. How different would it be if we use a `stub` validator instead of the `mock`:
{{< highlight kotlin >}}
class PassingValidator : Validator {
  override fun validate(query: String): Boolean {
    return true
  }
}
{{< /highlight >}}
5. What if we used an instance of the shipping `DefaultValidator` validator?

---

Let's get through the items of that list one by one.

#### 1. What's the relation between the shipping implementation of the Validator and the test-doubles? {#answer_1}
The only relation in this case are the types of the `Validator` being passed when constructing the instances of the collaborators it is injected into, as well as the return type of the call to the `validate` function. We don't have any behaviour-wise relations between the both. That brings us to the second question.

#### 2. How many tests would fail when a business rule for a valid query changes? {#answer_2}
From the answer of the previous question we conclude that not a single test will fail within the current constellation, which is way less than ideal. Clearly, we need to be informed when a behaviour changes (intentionally or unintentionally) to prevent regressions and bugs.

#### 3. How many tests should fail? {#answer_3}
As far as I can tell, there is no correct answer to this question, because it depends on the business rule that gets changed. What we can say with confidence is that we need to have an indication that something got changed - so we need at least one failing test. On the other hand, we want to have as little as possible amount of tests failing. We don't want the whole suite to fail.

#### 4. How different would it be if we use the `stub` validator instead of the `mock`? {#answer_4}
In this situation, there is no difference between using `stub` and `mock` in regards to the failing tests. There is no difference because when the business rule changes and we go and refactor the shipping code, we might **not** go and update all the places where we use a test-double to substitute the shipping implementation. So the test-doubles might not reflect the fact that the shipping implementation changed, hence the tests will keep passing.

#### 5. What if we used an instance of the shipping `DefaultValidator` validator? {#answer_5}
In this case, all tests where an instance of the shipping implementation of the validator is used would fail, and we'll need to go and fix all these tests. This is far better than having no test failures, but also going and updating lots of tests is not what we want to do either.

### Acceptance Test {#acceptance_test}
From the answers of the [2nd](#answer_2), [3rd](#answer_3) and the [5th](#answer_5) question we realise that we must find a way to catch possible regressions when a business rule changes (intentionally or unintentionally). Having a failing test as an indication is what we are striving for, but we don't want the whole test suite to fail and require us to go back and fix loads of tests. Thinking about the test pyramid, we would have a lot of micro tests, a bunch of acceptance tests and so on. Given the situation we described above, we might miss a regression by using the `test-doubles`, but surely we could catch that regression with an acceptance test. That way we would still have an indication in a form of a failing test for the regression, but in the same time we will keep the failing tests to minimum. The acceptance test is **always using real instances** of the whole structure being involved, except for the _boundaries_. For the _boundaries_ we use `test-doubles` in the acceptance test as well. The _boundaries_ are the points where our system has to talk to an external system, like a database or a network. But we realised that the `test-doubles` are not a silver bullet right? How do we make sure that the shipping implementation has the same behaviour as the `test-double`? Let's find out.

### Contract Tests
In the answers of the [1st](#answer_1) and the [4th](#answer_4) question and the discussion about the [Acceptance Test](#acceptance_test) we realise that no matter what kind of `test-double` we use, we have no relation in terms of behaviour between the shipping code and the `test-double` implementation established, and that's a bug problem. This is where the contract tests shine. The contract tests are the way to make sure that the behaviour of a `test-double` is in line with the behaviour of the shipping code. Check out the following video to see the contract tests in action:

{{< youtube S3qItwfhtCw >}}

### State or Behaviour?
An important topic at this point is to see what are we checking for. In the tests above, I didn't write a concrete validation line intentionally. One way to go is to **verify a behaviour**. That means that we can check if something is being called (additionally with expected arguments). Probably you've seen people complaining that checking behaviour leads to brittle tests which are getting coupled with the implementation details. Let's consider the situation from above - the `FindItems` or the `DeleteItems` classes. They call the `Repository` collaborator to _do the lookup_ or _do the cleanup_ respectively. There is no state these classes are dealing with, but there is an important behaviour: **do an action only if the validation passes**. In this case it makes perfect sense to check on the behaviour. We definitely need an indication in case that behaviour changes.

### Conclusion
Choosing what and how to test at a given situation is not an easy decision. Over the years I've realised that it takes lot of work and practice to build a good gut, and even then, there is no silver bullet or something like `the best approach`. We should constantly keep an eye at the process and keep balancing out.
