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
  val lookup = FindItems(validator, repository)
  given(validator.validate(query)).willReturn(true)

  lookup.search(query)

  validateTheOutputOfTheOperation
}

@Test
fun deletesMatches() {
  val validator = mock<Validator>()
  val cleanup = DeleteItems(validator, repository)
  given(validator.validate(query)).willReturn(true)

  cleanup.deleteMatches(query)

  validateTheOutputOfTheOperation
}
{{< /highlight >}}

In this case, we are using a `mock` validator, and we are setting it up to return a specific result.

Let's suppose that the shipping implementation of the validator looks like this:

{{< highlight kotlin >}}
class DefaultValidator: Validator {
  override fun validate(query: String): Boolean {
    return query.trim().length > 3
  }
}
{{< /highlight >}}

Now, here are a bunch of interesting questions:
1. What's the relationship between the shipping implementation of the Validator and the mocks?
2. How many tests would fail when the business rule for valid query changes (say from `length > 3` to `length > 8`)?
3. How many tests should fail?
4. How different would it be if we use a `stub` validator instead of the `mock`:
{{< highlight kotlin >}}
class PassingValidator : Validator {
  override fun validate(query: String): Boolean {
    return true
  }
}
{{< /highlight >}}
5. What if we used an instance of the shipping `DefaultValidator` in the tests?

---

Let's get through the items of that list one by one.

#### 1. What's the relationship between the shipping implementation of the Validator and the mocks? {#answer_1}
In this case, the relations are the type of the `Validator` being passed when constructing the collaborators' instances it is injected into and the return type of the `validate` function. We don't have any behavior-wise relations between both. That brings us to the second question.

#### 2. How many tests would fail when the business rule for valid query changes? {#answer_2}
From the previous question's answer, we conclude that not a single test will fail within the current constellation, which is way less than ideal. Clearly, we need to be informed if behavior changes (intentionally or unintentionally) to prevent regressions and bugs.

#### 3. How many tests should fail? {#answer_3}
As far as I can tell, there is no correct answer to this question because it depends on the business rule that gets changed. We can say with confidence that we need to indicate that something got changed, so we need at least one failing test. On the other hand, we want to have as little as possible amount of tests failing. We don't want the whole test-suite to fail.

#### 4. How different would it be if we use a `stub` validator instead of the `mock`? {#answer_4}
In this situation, there is no difference between using `stub` and `mock` in regards to the failing tests[^1]. When the business rule changes - we might go and change the shipping code. But we might **not** update all the places where we use a test-double to substitute the shipping implementation. So the test-doubles might not reflect that the shipping implementation changed. Hence the tests remain passing.

#### 5. What if we used an instance of the shipping `DefaultValidator` in the tests? {#answer_5}
In this case, all tests where an instance of the validator's shipping implementation is used would fail, and we'll need to go and fix all these tests. This is far better than having no test failures, but updating many tests is not what we want to do either.

### Acceptance Test {#acceptance_test}
From the answers of the [2nd](#answer_2), [3rd](#answer_3), and the [5th](#answer_5) question, we realize that we must find a way to catch possible regressions when a business rule changes (intentionally or unintentionally). Having a failing test as an indication is what we are striving for, but we don't want the whole test suite to fail and require us to go back and fix loads of tests. Thinking about the test pyramid, we would have many micro tests, a bunch of acceptance tests, and so on. Given the situation we described above, we might miss a regression using the `test-doubles` in the micro tests. But, we could catch that regression with an acceptance test. We would yet have an indication in the form of a failing test for the regression, but at the same time, we will keep the failing tests to a minimum. The acceptance test is **always using real instances** of the whole structure being involved, except for the _boundaries_. For the _boundaries_, we use `test-doubles` in the acceptance test as well. The _boundaries_ are the points where our system talks to an external system, like a database or a network. But we've seen that the `test-doubles` are not a silver bullet, right? How do we make sure that the shipping implementation has the same behavior as the `test-double`? Let's find out.

### Contract Tests
In the answers of the [1st](#answer_1) and the [4th](#answer_4) question and the discussion about the [Acceptance Test](#acceptance_test), we realize that no matter what kind of `test-double` we use - we have no relation in terms of behavior between the shipping code and the `test-double` implementation established. That's a big problem. This is where the contract tests shine. We use them to ensure that a `test-double` behavior is aligned with the shipping code's behavior. Check out the following video to see the contract tests in action:

{{< youtube S3qItwfhtCw >}}

### State or Behaviour?
An important matter at this point is to examine what we are checking for. In the tests above, I didn't write a concrete validation line intentionally. One way to go is to **verify a behavior**. That means that we can check if something is being called (additionally with expected arguments). You've probably seen people complaining that checking behavior leads to brittle tests that are getting coupled with the implementation details. Let's consider the situation from above - the `FindItems` or the `DeleteItems` classes. They call the `Repository` collaborator to _do the lookup_, or _do the cleanup_ respectively. There is no state these classes are dealing with, but there is a critical behavior: **do an action only if the validation passes**. In that case, it makes perfect sense to verify the behavior. We definitely need an indication in case that behavior changes.

### Conclusion
Choosing what and how to test in a given situation is not an easy decision. Over the years, I've realized that it takes a lot of work and practice to build a good gut, and even then, there is no silver bullet or something like `the best approach`. We should continuously keep an eye on the process and keep balancing it out.

[^1]: Considering JVM languages, namely Java or Kotlin, there are 2 differences between the mock and the stub: 1) Mocking the interface using a library like Mockito or Mockk comes with a considerable performance impact on the tests, so they will run way slower. 2) Having a stub will require a single change when we'll need to update it, and by using mocks, we'll need to go around the whole codebase and update the setup scripts, which means a lot more maintenance for the tests.
