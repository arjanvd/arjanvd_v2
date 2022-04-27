+++

title = "Creating a rainbow animation"
date = 2017-09-15T22:43:30+02:00 
tags = [ "code", "css", "html" ] 
series = [ "artikelen" ] 
draft = false
summary = "When I started thinking about a new logo for a new project, a rainbow was involved. I figured it would be nice to do an animation: stacking the seven colors one by one and then creating a subtle transition between the colors."

+++

_When I started thinking about a new logo for a new project, a rainbow was involved. I figured it would be nice to do an animation: stacking the seven colors one by one and then creating a subtle transition between the colours._

To be more precise: 

* the first colour starts the animation, and ends at the same time as the animation of the last color;
* when every block is in its place, the seven colours transform into a subtle gradient.

It should look something like this:

{{< image-gif src="23.gif" class="" alt="" figcaption="The rainbow anima.. Hey, what was that?" >}}

(If you don’t want to read the explanation below and want to go for the good stuff straight away: here’s the [Codepen](https://codepen.io/arjan/pen/YxNovm).)

### HTML

The HTML is rather straightforward:

```html
<div class="rainbow">
  <div class="rainbow__color color__red"></div>
  <div class="rainbow__color color__orange"></div>
  <div class="rainbow__color color__yellow"></div>
  <div class="rainbow__color color__green"></div>
  <div class="rainbow__color color__blue"></div>
  <div class="rainbow__color color__indigo"></div>
  <div class="rainbow__color color__violet"></div>  
</div>
```

### (S)CSS

I’m using SCSS here, for easy use of variables and the ability of creating mixins.

#### Colours

First, let’s create the colours of the rainbow as variables. I used [this image](https://www.img.webnots.com/2015/05/Rainbow-5.png), because I’m lazy and didn’t want to fire up Photoshop to extract the colors from an image. The variables then look like this:

```SCSS
$violet: #9400D3;
$indigo: #4B0082;
$blue: #00F;
$green: #0F0;
$yellow: #FF0;
$orange: #FF7F00;
$red: #F00;
```

#### Keyframes

For creating the animation, the parent container `<div class="rainbow"></div>` needs `position: relative;`. The child `<div class="rainbow__color"></div>` gets assigned `position:absolute`. That way, the children elements will get positioned relative to the parent container. In other words: they know where to start and stop (and – not unimportantly – stay still).

The animation is created by keyframes. The colour starts at the top and has no height (at 0%). It grows, and moves downwards until it reaches its final position and height (at 80%). In the last part the colour changes into a gradient (at 100%).

Each colour needs its own keyframe, because each is placed on a different position in the rainbow. The first colour (red) needs `top: 100%;` at 100% of the keyframe, while the seventh colour (violet) needs `top: 0;`. Every other colour needs to be positioned somewhere in between. There’s a little math involved to get this right for each color:

`distanceToTop = totalHeight - (totalHeight / totalColors) * numberOfColor`

For example: the third colour (yellow) has `distanceToTop` of `100% - (100% / 7) * 3 = 57.14286%`, so the keyframe becomes `100% { top: 57.14286%; }`. The code looks like this:

```SCSS
@keyframes move-yellow {
  0% { top: 0; height: 0; }
  80% { top: 100% - ((100% / 7) * 3); height: (100% / 7); background: $yellow; }
  100% { top: 100% - ((100% / 7) * 3); height: (100% / 7); background: linear-gradient($green 0% $yellow 15%); }
}
```

#### Animation

Furthermore, each colour needs its own animation, because each colour needs a different `animation-duration` and `animation-delay`. If every element was given the same duration and delay, all elements would start and finish animating at the same time, and that’s not what I meant to do. Each colour has to start at a set interval, but they have to end simultaneously. But how to calculate the different amounts of `animation-duration` and `animation-delay`?

##### Calculating animation-delay

The total duration of the complete rainbow animation is 2 seconds. In those 2 seconds, all seven colours have to animate, but they can’t start at the same time. In other words, they need a different `animation-delay`. The math looks like this:

`animationDelay = totalDuration * ((numberOfColor - 1) / totalColors)`

The first colour starts without delay, but the second colour has to wait 1/7th of 2 seconds, the third colour has to wait 2/7th of 2 seconds, et cetera.

##### Calculating animation-duration

The `animation duration` is the opposite of the animation delay.

`animationDuration = totalDuration - ((numberOfColor - 1) / totalColors) * totalDuration`

That means the first colour has 2 seconds to complete its animation (`2s - 0 = 2s`), the second colour has `2s - (1/7 * 2s) = 1.714286s`, the third colour has `2s - (2/7 * 2s) = 1.428572s`, et cetera.

##### Animation with properties

So, the animation of each colour looks like this:

```scss
.color__red { animation: move-red 2s ease-out forwards 0s; }
.color__orange { animation: move-orange (2s * (6 / 7)) ease-out forwards (2s * (1 / 7)); }
.color__yellow { animation: move-yellow (2s * (5 / 7)) ease-out forwards (2s * (2 / 7)); }
.color__green { animation: move-green (2s * (4 / 7)) ease-out forwards (2s * (3 / 7)); }
.color__blue { animation: move-blue (2s * (3 / 7)) ease-out forwards (2s * (4 / 7)); }
.color__indigo { animation: move-indigo (2s * (2 / 7)) ease-out forwards (2s * (5 / 7)); }
.color__violet { animation: move-violet (2s * (1 / 7)) ease-out forwards (2s * (6 / 7)); }
```

I used the shorthand notation. The order is as follows:

* the name of the keyframes-sequence the animation has to use;
* the duration of the animation;
* the style of the animation;
* the fill-mode of the animation (this has to be `forwards` so that it preserves its final state); and
* the delay of the animation

And now for the result!

<p data-height="349" data-theme-id="0" data-slug-hash="XazBeL" data-default-tab="result" data-user="arjan" data-embed-version="2" data-pen-title="Rainbow animation" class="codepen">See the Pen <a href="https://codepen.io/arjan/pen/XazBeL/">Rainbow animation</a> by Arjan (<a href="https://codepen.io/arjan">@arjan</a>) on <a href="https://codepen.io">CodePen</a>.</p><br />
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

### Hmmm, something’s wrong…

That’s almost what I had in mind, but something’s off. The last part doesn’t animate: there’s a sudden change between the colour and the gradient. Why isn’t this working?

#### Pseudo-element to the rescue

It turns out that gradients are not animatable. Luckily, [this article](https://medium.com/@dave_lunny/animating-css-gradients-using-only-css-d2fd7671e759) provided a solution: use an extra or a pseudo-element. When both elements are stacked upon each other, it’s possible to transition the opacity — which is animatable — of the top element. Brilliant. The keyframe looks as simple as it is:

```css
@keyframes gradient {
  0%   { opacity: 0; }
  100% { opacity: 1; }
}
```

I decided to use an pseudo-element, which looks like this in the (S)CSS:

```scss
.rainbow__color {
  position: absolute;
  width: 100%;
  &:before {
    content: '';
    display: block;
    width: 100%;
    height: 100%;
    opacity: 0; 
    animation: gradient 1s ease-out forwards 2s;    
  }
}
```

Setting `opacity: 0;` means it stays hidden until the animation is triggered. The animation has a duration of 1 second, and a delay of 2 seconds — which corresponds with the total duration of the movement of the colours.

#### Keyframes

Now that the change in colour to gradient has its own animation on a separate (pseudo-)element, it’s time to create the keyframe for the colours. It looks like this:

```css
@keyframes move-yellow {
  0%   { top: 0; height: 0; }
  20%  { height: (100% / 7) / 7; }
  100% { top: 100% - ((100% / 7) * 3); height: (100% / 7); }
}
```

(Notice I inserted a step at 20%. I thought that gave the animation a better effect. You may of may not disagree.)

### Result

Now it’s working like I wanted it to work as you can see in this Codepen.

<p data-height="370" data-theme-id="0" data-slug-hash="YxNovm" data-default-tab="result" data-user="arjan" data-embed-version="2" data-pen-title="Rainbow animation" class="codepen">See the Pen <a href="https://codepen.io/arjan/pen/YxNovm/">Rainbow animation</a> by Arjan (<a href="https://codepen.io/arjan">@arjan</a>) on <a href="https://codepen.io">CodePen</a>.</p><br />
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

Satisfying.
