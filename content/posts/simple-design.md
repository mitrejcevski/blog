---
title: "Simple Design - Separating responsibilities"
date: 2020-05-21T08:46:04+02:00
featured_image: "/images/simple_design/simple-design.jpg"
draft: true
---

Let's take a look at the following piece of code:

{{< highlight kotlin >}}
fun search(query: String): List<String> {
    val allItems: List<String> = ExternalSystem.fetchAllItems()
    val filtered = allItems.filter { it.contains(query) }
    val reordered = filtered.sortedWith(object : Comparator<String> {
        override fun compare(lhs: String, rhs: String): Int {
            if (lhs == query) {
                return -1
            } else if (rhs == query) {
                return 0
            }
            return lhs.compareTo(rhs)
        }
    })
    return reordered
}
{{< /highlight >}}

From a behavioral point of view, this code has no problems. Assuming it is correct, it will do its job, and everything seems OK.
However, from a design point of view, this code has a bunch of problems. Let's pass through them and see how we can tackle them down, ultimately improving the design.

#### Fetching
The first line is fetching items from an external system. Additionally, the call to the external system is static, which makes this code hard or maybe impossible to test. We'll look into the testability problem later in the article.

#### Filtering
In the following line, the results are being filtered out, keeping only the items where the `query` argument is fully contained.

#### Ordering
Finally, the results are being reordered alphabetically, moving the more exact results at the beginning of the list.


This function does at least three things: **fetching**, **filtering**, and **ordering**. Arguably, it touches different *levels of abstraction*. In particular, it delegates the **fetching** operation; it leverages the `filter` function from the SDK for the **filtering** operation, and it uses a custom `comparator` that does low-level operations inside for the **ordering**.

The first step towards improving this code's design is to split the different operations into different functions. Bear in mind that we always want to keep our steps small. We might end up with something like this:

{{< highlight kotlin >}}
fun search(query: String): List<String> {
    val allItems: List<String> = fetchAllItems()
    val filtered = filterItems(allItems, query)
    val reordered = reorderItems(filtered, query)
    return reordered
}

private fun fetchAllItems(): List<String> {
    return ExternalSystem.fetchAllItems()
}

private fun filterItems(
        items: List<String>,
        query: String
): List<String> {
    return items.filter { it.contains(query) }
}

private fun reorderItems(
        items: List<String>,
        query: String
): List<String> {
    return items.sortedWith(object : Comparator<String> {
        override fun compare(lhs: String, rhs: String): Int {
            if (lhs == query) {
                return -1
            } else if (rhs == query) {
                return 0
            }
            return lhs.compareTo(rhs)
        }
    })
}
{{< /highlight >}}

Now the operations are done by different functions. The `search` function became the so-called [**compose method**](https://scrutinizer-ci.com/docs/refactorings/compose-method), which only delegates the appropriate calls and eventually returns the result. Additionally, it operates at the same *level of abstraction*.
Sill, the class that holds these methods has at least three reasons to change. That violates the **Single Responsibility Principle**.

One reason to make a change in the class would be updating the call to the external system. Altering the search algorithm will add up another reason for changing the class. The third reason to make a change in the class would be changing the ordering algorithm. Let's fix that!

We can start by moving each different responsibility into a standalone component, and then inject those components as collaborators to this class. We may end up with something like this:

{{< highlight kotlin >}}
class Searcher(
        private val inventory: RemoteInventory,
        private val filter: ContainsFilter,
        private val order: KeywordMatchingOrder
) {

    fun search(query: String): List<String> {
        val allItems: List<String> = inventory.fetchAllItems()
        val filtered = filter.filterItems(allItems, query)
        val reordered = order.reorderItems(filtered, query)
        return reordered
    }
}
{{< /highlight >}}

We see an improvement, but we are not done yet. Now the class `Searcher` is injecting its collaborators through the constructor, and at this stage, they are concrete types. Even their names are referring to concrete implementations. Therefore, replacing an implementation will imply a change in this class again. It would be way better to depend on abstractions instead.

To do so, first, we will extract an interface from each concrete implementation:

{{< highlight kotlin >}}
interface Inventory {

    fun fetchAllItems(): List<String>
}

class RemoteInventory : Inventory {

    override fun fetchAllItems(): List<String> {
        return ExternalSystem.fetchAllItems()
    }
}
{{< /highlight >}}
---
{{< highlight kotlin >}}
interface Filter {

    fun filterItems(
            items: List<String>,
            query: String
    ): List<String>
}

class ContainsFilter : Filter {

    override fun filterItems(
            items: List<String>,
            query: String
    ): List<String> {
        return items.filter { it.contains(query) }
    }
}
{{< /highlight >}}
---
{{< highlight kotlin >}}
interface Order {

    fun reorderItems(
            items: List<String>,
            keyword: String
    ): List<String>
}

class KeywordMatchingOrder : Order {

    override fun reorderItems(
            items: List<String>,
            keyword: String
    ): List<String> {
        return items.sortedWith(object : Comparator<String> {
            override fun compare(lhs: String, rhs: String): Int {
                if (lhs == keyword) {
                    return -1
                } else if (rhs == keyword) {
                    return 0
                }
                return lhs.compareTo(rhs)
            }
        })
    }
}
{{< /highlight >}}

And then, we will replace the injected types in the class:

{{< highlight kotlin >}}
class Searcher(
        private val inventory: Inventory,
        private val filter: Filter,
        private val order: Order
) {

    fun search(query: String): List<String> {
        val allItems: List<String> = inventory.fetchAllItems()
        val filtered = filter.filterItems(allItems, query)
        val reordered = order.reorderItems(filtered, query)
        return reordered
    }
}
{{< /highlight >}}

Essentially, we have applied the **Open-Closed Principle** to the collaborators, and the **Dependency Inversion Principle** to the actual class.

At this stage, we ended up in a way better place. The collaborators are open for extension but closed for modification. The `Searcher` class is depending on abstractions instead of concrete types. Now, if we want to change the items loading, we can write a new implementation of the `Inventory` interface and provide that one when constructing the `Searcher` class. What if we need a filter that checks the exact matches of the keyword? - We will write a new implementation of the `Filter` type, and provide it in the constructor of the `Searcher.` The same applies if we want to change the ordering.

So, we can provide different implementations to the `Searcher` as long as they agree on the contract, and the `Searcher` doesn't care about the concrete implementations, aka *implementation details*.

#### Testing
Early on, we stated that the static call to the external system (in the first version of the code) makes it hard or maybe impossible to test. What are the problems with this call?
 - If the external call is fetching the results over the network, the test will depend on an active connection and the server that provides the results being up and running. Missing any of the conditions will make the test fail, so the test doesn't work 100% of the time. We end up with a *flaky* test.
 - If the external call is making calls to a database, despite the dependency on the database itself, it may corrupt the data in the database, causing lots of damage.
 - In both cases, the test might end up running slow.

All we need from the actual test for the `Searcher` is to verify its behavior, and to achieve it - we need data. We don't need to make calls to any external systems. We can set up the required data in memory and use it in the test. Given the `Inventory` abstraction that we came up with, setting up an in-memory implementation for the testing purposes is pretty straightforward.

#### Wrap up
The key for keeping the design simple is always in small things. It does take a little more time to keep it nice, but that's a really valuable long-term investment. It's almost like cutting a loan: we pay it monthly - we won't notice it. We stop paying for a while - we end up in a though situation. We need to be persistent in paying attention to small things and doing them right. There are situations when it feels like we can skip over a problem because it's easy to come back and change when needed, but usually that doesn't happen, so those small places pile up overtime into a bigger mess way more expensive to fix.
