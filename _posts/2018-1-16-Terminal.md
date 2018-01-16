---
layout: post
title: Qemu - Meson - Terminal Prompt
---

With meson 0.44 I had the problem, that when a test failed my terminal lost the prompt.  
I had to change the meson python code to get rid of this annoying problem.

`~/.local/bin/meson`
```
if __name__ == '__main__':
    sys.exit(main())
```

changed version :
`~/.local/bin/meson`
```
if __name__ == '__main__':
    out = main()
    os.system('stty sane')
    sys.exit(out)
```

This code sanitizes the tty before ending the meson command.  
And worked quite nice!
