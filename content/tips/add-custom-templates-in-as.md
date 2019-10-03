---
title: "Add custom live templates in Android Studio"
date: 2019-09-25T14:15:25+02:00
---

## What is a custom live template
In Android Studio, there is a concept of inserting a block of any textual content as a template wherever it is required, just by typing a keyword. By using live templates, we can insert frequently-used constructions into our code.

#### How is this going to be helpful?
Live templates are super handy to boost our speed while writing code. Overtime we use the same or similar constructions in our code like loops, conditions, declarations and even whole class templates (think of for example `RecyclerView.Adapter`, `RecyclerView.ViewHolder` and so on). Some folks are using live templates extensively when doing live-coding presentations. It saves a lot of time, plus it eliminates many possible mistakes.

## How to create a new live template

To configure live templates, we need to open the `Live Templates` page of the Android Studio settings: *Settings -> Editor -> Live Templates*. On the `Live Templates` page, we can see all the available live templates, edit them and create new templates.

{{< imgpreview src="/images/live_templates/live_templates_window.png" caption="Live templates window" >}}

To define a new template, on the right side of the window, click the **+** button. There are 2 options available: **Live Template** and **Template group**.

{{< imgpreview src="/images/live_templates/live_templates_new_group.png" caption="Define new live templates group" >}}

Go ahead and create a new group and name it **test**. This group will be supposed to hold live templates we will use for writing tests, so the name makes sense.

Next, we have to select the newly created group **test** and click on the **+** again to define new live template inside this group.

{{< imgpreview src="/images/live_templates/live_templates_new_template.png" caption="Define new live template" >}}

Once we select this option, on the bottom of the window we can see the live template editor

{{< imgpreview src="/images/live_templates/live_templates_template_setup.png" caption="Edit the live template" >}}

Here, the first thing we have to setup is the `abbreviation`. The abbreviation is something like a keyword that will trigger the insertion of the template inside the editor. We can also apply some description that could be handy, so for example if we have similar abbreviations for different templates, the description will be very helpful to select the correct template later when working with them.

For the sake of this example, let's put `test` as an abbreviation, and `JUnit test function` as a description.

Next, we have to define a context where our new template is going to be available. At the very bottom of the live template editor window, there is a little warning sign saying `No applicable context`

{{< imgpreview src="/images/live_templates/live_templates_context_undefined.png" caption="Undefined template context" >}}

Let's go ahead and define a context by clicking on the `Define` action.

{{< imgpreview src="/images/live_templates/live_templates_context_selection_popup_window.png" caption="Define context popup" >}}

As we can see in the image above, I've selected `Kotlin Class` as a template context, which means this template will be available in Kotlin class files.

Next, let's go ahead and set up the actual template that we want to be available for the given abbreviation. Let's apply the following code inside the `Template text:` field in the editor:

{{< highlight Kotlin >}}
@org.junit.jupiter.api.Test
fun $EXPR$() {
    org.junit.jupiter.api.Assertions.assertEquals($EXPR1$, $EXPR2$)
}
{{< /highlight >}}

There are additional setup options on the right side of the live template editor but we will ignore them for now. Finally, here is how the editor looks like:

{{< imgpreview src="/images/live_templates/live_templates_template_ready.png" caption="Live template setup" >}}

All we have to do next is just save this and we are done. A little note about the actual template code that we have applied - we are using fully-qualified names for the `Test` class and the `assertEquals()` method:

{{< highlight Kotlin >}}
org.junit.jupiter.api.Test
org.junit.jupiter.api.Assertions.assertEquals
{{< /highlight >}}

When we will use this template in the editor, Android Studio will automatically use imports and will prettify the code.
Let's see how does it look like. Open any Kotlin class file (because that's the context we have set for the template we've just created), and inside the class's body, type the corresponding template abbreviation: `test`

{{< imgpreview src="/images/live_templates/live_templates_usage_hint.png" caption="Use live template" >}}

As we can see, Android Studio pops up a little window where we can choose between the available templates. Just because on my side I have set a live template for JUnit4 test function beforehand, I can see both options available and I can choose which one I would like to use. Just by pressing {{< keystroke value="⏎" >}} *(Enter)* or {{< keystroke value="⇥" >}} *(Tab)* Android Studio will insert the selected template in the editor, and it will put the cursor at the first `$EXPR$` variable from the template (in this case the function name).

{{< imgpreview src="/images/live_templates/live_templates_template_inserted.png" caption="Live template inserted" >}}

Once we are done typing the name, we can press {{< keystroke value="⏎" >}} or {{< keystroke value="⇥" >}} again to jump to the next `$EXPR$` variable. Each {{< keystroke value="⏎" >}} or {{< keystroke value="⇥" >}} jumps to the next `$EXPR$` variable, and {{< keystroke value="⇧⇥" >}} *(Shift + Tab)* jumps to the previous.

It's important to note that when typing while the cursor is at specific `$EXPR$` variable, the typed text will appear at all of the variables with the very same name. In our example, we have 3 variables in total, and they all differ by adding a number at the end of them: `$EXPR$`, `$EXPR1$` and `$EXPR2$`. The actual numbers are not used as an order for the next/previous jump, rather, to differentiate them.

## Sharing live templates
Recently I've discovered there is an [open source repository](https://github.com/pranaypatel512/AndroidLiveTemplates) created by [Pranay Patel](https://twitter.com/pranaypatel_) with a bunch of cool live templates ready to be used. This repository is welcoming pull requests so anyone can contribute and add some more live templates.

That's all for now, hope you find this useful! Feel free to [follow me on Twitter](https://twitter.com/jovchem) so you won't miss new content.

{{< twitterfollow account="jovchem">}}
