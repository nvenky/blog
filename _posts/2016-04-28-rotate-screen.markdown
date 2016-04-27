---
layout: post
title:  "Espresso - Testing persisted state by rotating screen"
subtitle: "Rotate screen"
date:   2016-04-27 20:01:01
categories: [android]
---

By rotating the screen in Espresso, we can quickly test whether the activity persists the state of the elements and restores it correctly.

**NOTE:** Android would destroy the existing activity and create a new one. So we need to find the new activity and update the reference of activity to proceed further in the test. The code below does the same.

## BaseTest.java

{% highlight java %}
private Activity activity;

protected T rotateScreen() {
    unregisterIdlers();
    int newOrientation = currentOrientation() == ActivityInfo.SCREEN_ORIENTATION_PORTRAIT ? ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE : ActivityInfo.SCREEN_ORIENTATION_PORTRAIT;
    activity.setRequestedOrientation(newOrientation);
    try {
        Thread.sleep(500);
    } catch (InterruptedException e) {
        fail("Failed to wait for orientation change");
    }
    activity = getNewInstanceOfActivity();
    return activity;
}

protected T getNewInstanceOfActivity() {
    final CountDownLatch latch = new CountDownLatch(1);
    final List<Activity> activities = new ArrayList<>();
    getActivity().runOnUiThread(new Runnable() {
        @Override
        public void run() {
            final ActivityLifecycleMonitor monitor = ActivityLifecycleMonitorRegistry.getInstance();
            if (monitor.getActivitiesInStage(Stage.DESTROYED).size() == 0) {
                fail("Expected an activity to be in Destroyed state");
            }
            activities.addAll(monitor.getActivitiesInStage(Stage.RESUMED));
            latch.countDown();
        }
    });
    try {
        latch.await(3, TimeUnit.SECONDS);
    } catch (InterruptedException e) {
        fail("Unable to get the New Activity");
    }
    return (T) activities.get(0);
}

{% endhighlight %}
