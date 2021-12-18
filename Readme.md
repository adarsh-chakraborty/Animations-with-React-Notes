# Animating React Apps and Components

We are going to explore css transitions and animations and some libraries to animate our react apps.

When working with animations in react apps, we got multiple ways to do that, the simplest and straight forward way is to use css transitions.

## Using CSS transitions

Let's take an example, we want an animation like
Fly in from top and also change the opacity, same when closing it.

**Open and closed classes:**

```css
.ModalOpen {
  opacity: 1;
  transform: translateY(0);
}

.ModalClosed {
  opacity: 0;
  transform: translateY(-90px);
}
```

When open class is applied, it will translate to `0` (original position set in html)
and it's opacity will be set to 1 (100% visible).

To see it in action smoothly, we need to apply transition property to the element.

```css
.Modal {
  transition: all 250ms ease-out;
}
```

## CSS Animations

CSS Animations simply allows us to make more complex and controlled animations than just transition.

Because with just transition, we're just saying _"Hey, whenever any property changes, animate that change, and figure out how to animate it on your own basically."_

Now an animation in CSS is detailed described set of steps you want to execute, for that we create `keyframes`

To create keyframes, we need to write `@keyframes` followed by the name of the keyframe, and inside it we can define the steps.

```css
@keyframes openModal {
  0% {
    opacity: 0;
    transform: translateY(-100%);
  }
  50% {
    opacity: 1;
    transform: translateY(20%);
  }
  100% {
    opacity: 1;
    transform: translateY(0%);
  }
}
```

Here, we described states like `0%`,`50%` and `100%`, and we can use normal css to tell what it should look like at that point of time.

Now that we have setup the keyframes, it's time to apply it to our element class.

So in ModalOpen & ModalClose classes:

```css
.ModalOpen {
  animation: openModal 0.4s ease-out forwards;
}
```

- `animation` is a css property.
- `openModal` is the name of the keyframe.
- `0.4s` duration of overall animation.
- `ease-out` Defines how it moves between Indvidual steps/state of keyframes.
- `forwards` keeps the final keyframe value, if we don't define it, it will jump back to starting state of animation.

Let's do it for close modal, but in reverse order.

```css
@keyframes closeModal {
  0% {
    opacity: 1;
    transform: translateY(0);
  }
  50% {
    opacity: 0.7;
    transform: translateY(-60px);
  }
  100% {
    opacity: 0;
    transform: translateY(-90px);
  }
}

.ModalClosed {
  animation: closeModal 0.4s ease-out forwards;
}
```

## Limitations of CSS animations and transition

One drawback is, we are always setting opacity and position, it's always exists in the dom but just not visible.

That means, The html is Populating on the dom, Just invisible due to opacity and If it's alot can slow down the app.

Also, adding the element conditionally would work because we just adding css class but removing it wouldn't work because it removes the element instantly, It doesn't wait for JSX for re-rendering and wait for animation to finish because it's isn't aware of the animation.

---

# React Transition Group

ReactTransitionGroup is a third party library. It exposes transition stages, manages classes and group elements and manipulates the DOM in useful ways, making the implementation of actual visual transitions much easier.
## Installation
```
npm install react-transition-group --save
```

## Using the Transition component

Transition component is one of the important components that this package exports. As it is a default export we can name it anything but `Transition` is preferred.

**Importing the component:**

```javascript
import Transition from 'react-transition-group/Transition';
```

Now, we just need to wrap the html we need to animate with this Transition component.

```javascript
<Transition>
          <div
            style={{
              backgroundColor: 'red',
              width: 100,
              height: 100,
              margin: 'auto'
            }}
          ></div>
        </Transition>
```

That alone, won't do much, now we have to pass certain props to the transition element.

- `in` Determines should be shown or not.
- `timeout` value determines the duration of the animation. `{300}` is equal to `300ms`.
- `mountOnEnter` It's a boolean prop.
- `unmountOnExit` Removes the html from dom, It's also boolean prop.

#### Transition manages 4 internal states of transition for us. 

- `ENTERING`
- `ENTERED`
- `EXITING`
- `EXITED`

We can listen to such states and manage how our element should be shown, Let's see how to do it.

Flow of state: 
```
ENTERING -> ENTERED
EXITING -> EXITED
```

So, we can now take advange of these states through the function.
First bind the `in` to some prop, `timeout` determines how long the animation should be played.

```javascript
<Transition in={this.state.showBlock} timeout={300}>
          {
           (state) => () 
          }
        </Transition>
```
Here, in the transition component, we can write a function inside `{}` that receives the state, it will be the one of the internal state that transition component manages for us and the returned component will be rendered to dom.

```javascript
<Transition
          in={this.state.showBlock}
          timeout={1000}
          mountOnEnter
          unmountOnExit
        >
          {(state) => (
            <div
              style={{
                backgroundColor: 'red',
                width: 100,
                height: 100,
                margin: 'auto',
                transition: 'opacity 1s ease-out',
                opacity: state === 'exiting' ? 0 : '1'
              }}
            ></div>
          )}
        </Transition>
```

If a component has to be unmounted && we're animating suppose changing opacity on `exited`, the transition will be too late and won't be visible, so instead we have to target `exiting` state, so we can see the animation as well as remove it from the dom.

So, the element won't be unmounted until the animation has finished.



