# Rusty tresures

My collection of treasures from the Rust ecosystem.

<https://srgsanky.github.io/rusty-treasures/>


## Command to add new post

```bash
hugo new content content/posts/making-time-human.md
```

## Internals

### Layout

`./layouts/_default/single.html` is mostly a copy of
<https://github.com/theNewDynamic/gohugo-theme-ananke/blob/a0019812ff62d978781392c765addbaee1c1eddf/layouts/_default/single.html#L53>

In order to increase the width of the post's content, I changed `w-two-thirds-l` to `w-100-l` as suggested in
<https://github.com/theNewDynamic/gohugo-theme-ananke/issues/288>

