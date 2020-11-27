---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults
layout: default
isHome: true
---

<div class="home">

  <h1 class="centered large-text">
      <div id="typed-strings"></div>
      <span id="typed"></span>
  </h1>

  <script>
      var typed = new Typed('#typed', {
        strings: ["Hi, I'm a software engineer 👋", "Hire me to translate your ideas into code!"],
        smartBackspace: true,
        typeSpeed: 50,
        backSpeed: 30,
        backDelay: 1500,
        loop: true
      });
  </script>
</div>