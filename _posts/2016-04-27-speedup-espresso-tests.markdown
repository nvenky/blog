---
layout: post
title:  "Speed up Espresso Tests"
subtitle: "Idling resource polling"
date:   2016-04-27 19:01:01
categories: [android]
---

You can write reliable tests in Espresso by using Idling resources. But when we were running the tests, it was taking at least 5 seconds to check whether the Idling resource has become idle. Overall build time increased drastically when we started adding more tests.

Ideal way is to notify Espresso that the resource is now ready in the Idling resource but we had to implement that logic in all the UI components that were used in the Idling resources. We looked at the internals of Espresso to see whether there is a quick way to decrease the polling time. In IdlingPolicies, "dynamicIdlingResourceWarningPolicy" field was configured to poll the Idling resource every 5 Seconds and write a warning in the log if it is still busy. In our case, this polling was notifying Espresso that resources have turned idle and the test can proceed.

We changed this idleTimeout from 5 SECONDS to 200 MILLISECONDS and immediately tests started running really fast. We cut down the build time from 11 minutes to 3 minutes with this simple change.

## BaseTest.java

{% highlight java %}

@BeforeClass
public static void changeEspressoIdlingTimeout() throws Exception {
    Field warningPolicyField = IdlingPolicies.class.getDeclaredField("dynamicIdlingResourceWarningPolicy");
    warningPolicyField.setAccessible(true);
    IdlingPolicy policy = (IdlingPolicy) warningPolicyField.get(null);
    setValue(policy, "idleTimeout", (long) 200);
    setValue(policy, "unit", TimeUnit.MILLISECONDS);
}

private static void setValue(IdlingPolicy policy, String fieldName, Object value) throws NoSuchFieldException, IllegalAccessException {
    Field field = IdlingPolicy.class.getDeclaredField(fieldName);
    field.setAccessible(true);
    field.set(policy, value);
}

{% endhighlight %}

**Each UI interaction now takes 200 MILLISECONDS instead of standard 5 SECONDS.**
