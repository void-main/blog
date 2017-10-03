---
layout: post
comments: true
title:  "How to Use New Android Animation API for Compat Fragment"
date:   2013-08-22 19:55:32
tags: android animator compat fragment
categories: android animator fragment
---

## Why
Since honeycomb (API level 11), Google has introduced a new set of APIs to help developers build better app, including [Fragment](http://developer.android.com/guide/components/fragments.html) and [property animation](http://developer.android.com/guide/topics/graphics/prop-animation.html).

These new APIs are great, but they are not quite compatible with older devices, thus Google provided a [support library](http://developer.android.com/tools/support-library/index.html) to solve this problem.

While building my app, I'm trying to use the [compat fragment](http://developer.android.com/reference/android/support/v4/app/Fragment.html) to support as many devices as possible, meanwhile, I want to take advantage of new set of animation apis to create animation easier.

The [FragmentTransaction](http://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html) class has a simple API named `setCustomAnimations()` to set the animation while switching fragments. But, sadly, when I tried to use the animator xml file (I mean, using property animation elements like `<objectAnimator>` in the animation xml) on Gingerbread emulator (with API level less than 11), the app simply crashed complaining it does not understand the `objectAnimator` sh*t, which means you have to give up on the new animation API with compat fragment.

No way!

And finally, I managed to find a way to get this work. Here's what I've tried.

If you are urged to see a demo, please check the [readme of this sample](https://github.com/void-main/FragmentCompatAnimator) on github.

## Starting with a simple fragment
Since this is a experimental project, I don't want to build some complex fragments, that's why I decided to use [RoundedColourFragment](https://github.com/JakeWharton/ActionBarSherlock/blob/master/actionbarsherlock-samples/styled/src/com/actionbarsherlock/sample/styled/RoundedColourFragment.java) from [ActionBarSherlock](http://actionbarsherlock.com/) project.

The fragment is quite simple, it will generate a colored rounded rectangle as the view of the fragment, whose color can be specified as a argument of the constructor.

## Importing NineOldAndroids
In order to use the new animation api on older versions of android, I found of the [NineOldAndroids](http://nineoldandroids.com/) project. 

Even though I didn't quite get its name (sad), the project is really awesome. It somehow managed to give the developer a way to use the new animation apis even on Android 1.0, amazing. It's fun, but the only way is you cannot put them in XML files, still need to work out how to use the animation.

## Two parts of the animation
Not being able to use the animation in XML files, I have to write some code to make this happen. Here's a methods I defined to trigger the new animation.

```
import com.nineoldandroids.animation.Animator;
import com.nineoldandroids.animation.AnimatorListenerAdapter;
import com.nineoldandroids.animation.ObjectAnimator;

……

private void flipit() {
	ObjectAnimator visToInvis = ObjectAnimator.ofFloat(frontFrag.getView(), 
	    "rotationY", 0f, 90f);
	visToInvis.setDuration(300);
	visToInvis.setInterpolator(accelerator);
	visToInvis.addListener(new AnimatorListenerAdapter() {
    	@Override
    	public void onAnimationEnd(Animator anim) {
    		FragmentTransaction ft = getSupportFragmentManager()
				.beginTransaction();
			ft.replace(R.id.root, backFrag);
			ft.commit();
    	}
    });
    visToInvis.start();
}
```

This is the first part of the animation, the old fragment fliped to disappear by changing the `rotationY` property from 0 to 90 degree. Once this animation is finished, I need to replace the old fragment with the new one by calling `FragmentTransaction#replace` method.

But we cannot simply let the new fragment appear and be there, it will be awkward. So, I've designed a callback that will be triggered once the new fragment's view is ready, the code is listed below.

```
@Override
public void onViewCreated(View view, Bundle savedInstanceState) {
	super.onViewCreated(view, savedInstanceState);
	
	if (mListener != null) {
		mListener.onViewCreated();
	}
}
```

Once the view is created, I'll kick off the second part of the animation, which changes the `rotationY` property from -90 degree to 0. Here's the code.

```
@Override
public void onViewCreated() {
	View v = backFrag.getView();
	final ObjectAnimator invisToVis = ObjectAnimator.ofFloat(v, "rotationY",
            -90f, 0f);
    invisToVis.setDuration(300);
    invisToVis.setInterpolator(decelerator);
	
	invisToVis.start();
}
```

And now, the animation is working perfectly.

## Show me the code
I've created a project on github, called [FragmentCompatAnimator](https://github.com/void-main/FragmentCompatAnimator), make sure to check it out.

## Conclusion
It is always a good idea to use new APIs for better performance and use experience. But compatibilty is a tough problem. Thanks to all opensource framework contributors, it is and will be easier for us developers to take advantage of the new APIs but still be able to make it work on older devices.
