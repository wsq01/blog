

## 按钮
```html
<button
  data-text="Start"
  class="btn btn-primary btn-ghost btn-border-stroke  btn-text-float-up"
>
  <div class="btn-borders">
    <div class="border-top"></div>
    <div class="border-right"></div>
    <div class="border-bottom"></div>
    <div class="border-left"></div>
  </div>
  <span class="btn-text">Start</span>
</button>
```
```less
.btn {
  --hue: 190;
  --ease-in-duration: 0.25s;
  --ease-in-exponential: cubic-bezier(0.95, 0.05, 0.795, 0.035);
  --ease-out-duration: 0.65s;
  --ease-out-delay: var(--ease-in-duration);
  --ease-out-exponential: cubic-bezier(0.19, 1, 0.22, 1);
  position: relative;
  padding: 1rem 3rem;
  font-size: 1rem;
  line-height: 1.5;
  color: white;
  text-decoration: none;
  background-color: hsl(var(--hue), 100%, 41%);
  border: 1px solid hsl(var(--hue), 100%, 41%);
  outline: transparent;
  overflow: hidden;
  cursor: pointer;
  user-select: none;
  white-space: nowrap;
  transition: 0.25s;

  &:hover {
    background: hsl(var(--hue), 100%, 31%);
  }

  &-primary {
    --hue: 171;
  }

  &-ghost {
    color: hsl(var(--hue), 100%, 41%);
    background-color: transparent;
    border-color: hsl(var(--hue), 100%, 41%);

    &:hover {
      color: white;
    }
  }

  &-border-stroke {
    border-color: hsla(var(--hue), 100%, 41%, 0.35);

    .btn-borders {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;

      .border-top {
        position: absolute;
        top: 0;
        width: 100%;
        height: 1px;
        background: hsl(var(--hue), 100%, 41%);
        transform: scaleX(0);
        transform-origin: left;
      }

      .border-right {
        position: absolute;
        right: 0;
        width: 1px;
        height: 100%;
        background: hsl(var(--hue), 100%, 41%);
        transform: scaleY(0);
        transform-origin: bottom;
      }

      .border-bottom {
        position: absolute;
        bottom: 0;
        width: 100%;
        height: 1px;
        background: hsl(var(--hue), 100%, 41%);
        transform: scaleX(0);
        transform-origin: left;
      }

      .border-left {
        position: absolute;
        left: 0;
        width: 1px;
        height: 100%;
        background: hsl(var(--hue), 100%, 41%);
        transform: scaleY(0);
        transform-origin: bottom;
      }

      // when unhover, ease-out left, bottom; ease-in right, top

      .border-left {
        transition: var(--ease-out-duration) var(--ease-out-delay)
          var(--ease-out-exponential);
      }
      .border-bottom {
        transition: var(--ease-out-duration) var(--ease-out-delay)
          var(--ease-out-exponential);
      }

      .border-right {
        transition: var(--ease-in-duration) var(--ease-in-exponential);
      }
      .border-top {
        transition: var(--ease-in-duration) var(--ease-in-exponential);
      }
    }

    &:hover {
      color: hsl(var(--hue), 100%, 41%);
      background: transparent;

      .border-top,
      .border-bottom {
        transform: scaleX(1);
      }
      .border-left,
      .border-right {
        transform: scaleY(1);
      }

      // when hover, ease-in left, bottom; ease-out right, top

      .border-left {
        transition: var(--ease-in-duration) var(--ease-in-exponential);
      }
      .border-bottom {
        transition: var(--ease-in-duration) var(--ease-in-exponential);
      }

      .border-right {
        transition: var(--ease-out-duration) var(--ease-out-delay)
          var(--ease-out-exponential);
      }
      .border-top {
        transition: var(--ease-out-duration) var(--ease-out-delay)
          var(--ease-out-exponential);
      }
    }
  }

  &-text-float-up {
    &::after {
      position: absolute;
      content: attr(data-text);
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      opacity: 0;
      transform: translateY(35%);
      transition: 0.25s ease-out;
    }

    // when hover, ease-in top-text; ease-out bottom-text

    .btn-text {
      display: block;
      transition: 0.75s 0.1s var(--ease-out-exponential);
    }

    &:hover {
      // when hover, ease-in bottom-text; ease-out top-text

      .btn-text {
        opacity: 0;
        transform: translateY(-25%);
        transition: 0.25s ease-out;
      }

      &::after {
        opacity: 1;
        transform: translateY(0);
        transition: 0.75s 0.1s var(--ease-out-exponential);
      }
    }
  }
}
```
{% asset_img gif1.gif %}

## 文字
```html
<header>
  <h1 class="title slide-bar">I'm alphardex.</h1>
  <p class="subtitle slide-bar">A CSS Wizard</p>
</header>
```
```less
.slide-bar {
  position: relative;
  color: transparent;
  animation: fill-text-white 2s 1.6s forwards;

  &::before {
    position: absolute;
    content: "";
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: #35b9f1;
    transform: scaleX(0);
    transform-origin: left;
    animation: slide-in-out 2s cubic-bezier(0.75, 0, 0, 1) forwards;
  }
}

@keyframes slide-in-out {
  50% {
    transform: scaleX(1);
    transform-origin: left;
  }

  50.1% {
    transform-origin: right;
  }

  100% {
    transform: scaleX(0);
    transform-origin: right;
  }
}

@keyframes fill-text-white {
  to {
    color: white;
  }
}

header {
  .title,
  .subtitle {
    width: 250px;
    height: 30px;
  }

  .title {
    margin: 0;
    font-family: Lora, serif;
    font-size: 32px;
    line-height: 30px;

    &::before {
      background: #FF4081;
    }
  }

  .subtitle {
    margin: 10px 0 0 0;
    font-family: Lato, sans-serif;
    font-size: 12px;
    line-height: 30px;
    letter-spacing: 5px;
    text-transform: uppercase;
    animation-delay: 3.2s;

    &::before {
      background: #03A9F4;
      animation-delay: 2s;
    }
  }
}
```
{% asset_img gif2.gif %}