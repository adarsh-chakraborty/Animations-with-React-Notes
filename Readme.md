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

## Animation Timings

What if we want to setup different timings for different states, like for Entering, Exiting etc.

For that, we can send an object with two keys, `enter` and `exit` and pass it to `timeout` prop in Transition.

```javascript
const animationTimings = {
    enter: 400,
    exit: 1000,
}

<Transition timeout={animationTimings}>
<p>Hello</p>
</Transition>
```

Also note, the css timings should be same as well for this to work.

## Transition Events

Sometimes, we might want to execute some code when certain state changes in the Transition, not just render some JSX, for that we got various callbacks.

We have 6 different events we can listen to, on a Transition component.

These will be executed as the following order.

- `onEnter={() => {}}`
- `onEntering={() => {}}`
- `onEntered={() => {}}`
- `onExit={() => {}}`
- `onExiting={() => {}}`
- `onExited={() => {}}`

We can bound a function to these events, which will be executed in the respective order as shown above.

```javascript
<Transition 
in={state.show}
timeout={animationTimings}
mountOnEnter
unmountOnExit
onEnter={() => console.log('onEnter')}
onEntering={() => console.log('onEntering')}
onEntered={() => console.log('onEntered')}
onExit={() => console.log('onExit')}
onExiting={() => console.log('onExiting')}
onExited={() => console.log('onExited')}
>
{state => <p>My JSX Code to Render hehe</p>}
</Transition>
```

## CSS Transition Component

Till now, we are manually checking the state of animation and based on the state we are attaching css class to the element, but Sometimes we might want to have pre-defined css classes depending on the state of Animation.

Turns out, the package we're using have a component for just that! It's imported from `"react-transition-group/CSSTransition"`

CSSTransition does not use any function in-between the tags, Instead we just write the JSX code, you want to output.

**Important:**
CSSTransition requires a prop called `classNames`, yes `classNames` and not ~className~.

**What does it do?**  
Whatever className we have on the wrapped element, It will keep em and merge the conditional classNames into the existing ones.

**The new classes**  
We can define the trunk of these classes on the prop.

Example: `classNames="fade-slide"`

We named it `fade-slide`, we can name it whatever we want, Now what it will do is-  
The CSSTransition component will automatically cycle through a couple of css-classes  and merge them to the className of wrapped element. 

The Classes it cycles through are `your-trunk-<state>`.

In our case, It will cycle through -

- `fade-slide-enter` attached at right at the point of start.
- `fade-slide-enter-active` Which will be kept attached while it is in the entering mode. (Duration of timeout)
- `-enter` class will be removed right after one frame.
- `fade-slide-exit` will be attached when we start exiting.
- `fade-slide-exit-active` it will be attached while exiting.

So now, all we have to do is to write all these classes in the css.


```javascript
import CSSTransition from "react-transition-group/CSSTransition";

const Modal = props => {
    return (
        <CSSTransition>
        mountOnEnter
        unmountOnExit
        in={props.show}
        timeout={animationTimings}
        classNames="fade-slide">

        <div className="Modal">
            <h1>Bawwww</h2>
            <p>That wasn't me barking, promise</p>
        </div>
        </CSSTransition>
    )
}
```

```css

.fade-slide-enter-active {
  animation: openModal 0.4s ease-out forwards;
}

.fade-slide-exit-active {
  animation: closeModal 1s ease-out forwards;
}

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

```

### Customizing CSS classNames

We had a look in CSSTransition component, how it uses a base class-name and it automatically expands this classnames according to the various states of animation such as `-exit` and `-active` and so on.

But sometimes we want to use our defined classNames, so essentially we don't have to define `-enter` `-active` classNames, for that to work...

We need to pass an object to the `classNames`, and in this object we can define the classNames that it will use for various states of aniamtion.

So, this object should have certain properties.
```javascript
const modalClass = {
    enter: '', // nothing
    enterActive: 'ModalOpen',
    exit: '',
    exitActive: 'ModalClosed'
    /* there are 2 more
    appear:
    appearActive: */ 
}
```

- `appear` and `appearActive` is used for something which is first time rendered on the dom.
- Not when rendering it conditionally, but instead when the application loads for the first time. (Hard coded Html)
- So if the application loads and it's always there (hard coded), `appear` will be used when it's first time mounted on the dom.

Now, we can simply pass such object to the classNames prop in CSSTransition component.

```javascript
 <CSSTransition>
        mountOnEnter
        unmountOnExit
        in={props.show}
        timeout={animationTimings}
        classNames= {
          enter: '', // nothing
          enterActive: 'ModalOpen',
          exit: '',
          exitActive: 'ModalClosed'
        }>

        <div className="Modal">
            // html here
        </div>
</CSSTransition>
```

## Animating Lists

We cannot animate Lists with just CSS or Transition component, for this we have another package exported from `"react-transition-group/TransitionGroup`.

We use this `TransitionGroup` element where we output list or dynamic list, By default it renders a div at it's place, so we can use a `component` property 

- PROPERTY = PROP(S) OK 

So, to render an un-ordered list instead of a div, we have to set prop `component="ul"` on TransitionGroup.

**Using the TransitionGroup:**

TransitionGroup can be used to animate List, but it works only in conjunction with 'Transition' or 'CSSTransition' components. So we will have to wrap our list-items in one of those components.


```javascript
import TransitionGroup from 'react-transition-group/TransitionGroup';
import CSSTransition from 'react-transition-group/CSSTransition';

const ListItems = props => {

    return (
        <CSSTransition 
        key={props.index}
        classNames="fade"
        timeout={300}>
            
            <li>{props.item}</li>
        </CSSTransition>
    )
}

const List = props => {
    // Map ListItems to create dynamic list
     const listItems = state.items.map((item,index) => (
             <ListItems index={index} item={item}>
        ));

    return (
        <TransitionGroup 
        component="ul"
        className="List">
        {listItems}
        </TransitionGroup>
    )
}


```

The special thing about `TransitionGroup` is that it is able to handle multiple items, It determies the changes, if it's removed or added, It will then manually set the `in` property in the wrapped `Transition` or `CSSTransition` property so we don't have to do it ourselves. 


As we set the css trunk `fade`, Now we can define the css trunks -

```css
.fade-enter{
    opacity: 0;
}

.fade-enter-active{
    opacity: 1;
    transition: opacity 0.3s ease-out;
}

.fade-exit{
    opacity: 1;
}

.fade-exit-active{
    opacity: 0;
    transition: opacity 0.3s ease-out;
}
```

## Alternative Packages for React Animations

There are various libraries for animations in React.  

Some popular Alternatives are -  

- **React Motion**
- **React Move**
- **React Router Transition**
 

