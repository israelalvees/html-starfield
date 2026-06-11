<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>SISWEB Background</title>

  <style>
    html, body {
      margin: 0;
      padding: 0;
      overflow: hidden;
      height: 100%;
      width: 100%;
      background: #F4F5F3;
    }

    canvas {
      position: absolute;
      top: 0;
      left: 0;
      display: block;
    }
  </style>
</head>

<body>

<canvas id="starfield"></canvas>

<script>

  // Configurações padrão
  let NUM_PARTICLES = 100;
  let SPEED = 0.5;
  let LIFETIME = 200;

  // Cores padrão SISWEB
  let COLOR_PALETTE = [
    "#23964B",
    "#2AA354",
    "#A8C7BC",
    "#C88235"
  ];

  let MIN_SIZE = 1;
  let MAX_SIZE = 3;

  // Leitura dos parâmetros da URL
  const params = new URLSearchParams(window.location.search);

  if (params.has("num_particles")) {
    NUM_PARTICLES = parseInt(params.get("num_particles"));
  }

  if (params.has("speed")) {
    SPEED = parseFloat(params.get("speed"));
  }

  if (params.has("lifetime")) {
    LIFETIME = parseInt(params.get("lifetime"));
  }

  if (params.has("color_palette")) {
    COLOR_PALETTE = params
      .get("color_palette")
      .split(",")
      .map(c => `#${c.trim()}`);
  }

  if (params.has("min_size")) {
    MIN_SIZE = parseFloat(params.get("min_size"));
  }

  if (params.has("max_size")) {
    MAX_SIZE = parseFloat(params.get("max_size"));
  }

  const canvas = document.getElementById("starfield");
  const ctx = canvas.getContext("2d");

  let w;
  let h;
  let particles = [];

  class Particle {

    constructor() {
      this.reset();
    }

    reset() {

      this.x = Math.random() * w;
      this.y = Math.random() * h;

      const angle = Math.random() * Math.PI * 2;
      const speed = SPEED * (0.5 + Math.random());

      this.vx = Math.cos(angle) * speed;
      this.vy = Math.sin(angle) * speed;

      this.size =
        MIN_SIZE +
        Math.random() * (MAX_SIZE - MIN_SIZE);

      this.age =
        Math.floor(Math.random() * LIFETIME);

      this.color =
        COLOR_PALETTE[
          Math.floor(
            Math.random() * COLOR_PALETTE.length
          )
        ];
    }

    update() {

      this.x += this.vx;
      this.y += this.vy;
      this.age++;

      if (
        this.age > LIFETIME ||
        this.x < 0 ||
        this.x > w ||
        this.y < 0 ||
        this.y > h
      ) {

        this.age = 0;

        this.x = Math.random() * w;
        this.y = Math.random() * h;

        const angle =
          Math.random() * Math.PI * 2;

        const speed =
          SPEED * (0.5 + Math.random());

        this.vx = Math.cos(angle) * speed;
        this.vy = Math.sin(angle) * speed;

        this.size =
          MIN_SIZE +
          Math.random() *
          (MAX_SIZE - MIN_SIZE);

        this.color =
          COLOR_PALETTE[
            Math.floor(
              Math.random() *
              COLOR_PALETTE.length
            )
          ];
      }
    }

    draw() {

      const halfLife = LIFETIME / 2;

      let alpha;

      if (this.age <= halfLife) {
        alpha = this.age / halfLife;
      } else {
        alpha =
          1 -
          (this.age - halfLife) /
          halfLife;
      }

      ctx.fillStyle =
        this.hexToRgba(
          this.color,
          alpha
        );

      ctx.beginPath();
      ctx.arc(
        this.x,
        this.y,
        this.size / 2,
        0,
        Math.PI * 2
      );

      ctx.fill();
    }

    hexToRgba(hex, alpha) {

      const bigint =
        parseInt(hex.slice(1), 16);

      const r = (bigint >> 16) & 255;
      const g = (bigint >> 8) & 255;
      const b = bigint & 255;

      return `rgba(${r},${g},${b},${alpha})`;
    }
  }

  function resize() {

    w = canvas.width = window.innerWidth;
    h = canvas.height = window.innerHeight;

    particles = [];

    for (
      let i = 0;
      i < NUM_PARTICLES;
      i++
    ) {
      particles.push(
        new Particle()
      );
    }
  }

  function animate() {

    // ALTERAÇÃO PRINCIPAL:
    // preto -> branco acinzentado

    ctx.fillStyle = "#F4F5F3";

    ctx.fillRect(
      0,
      0,
      w,
      h
    );

    for (let p of particles) {
      p.update();
      p.draw();
    }

    requestAnimationFrame(
      animate
    );
  }

  window.addEventListener(
    "resize",
    resize
  );

  resize();
  animate();

</script>

</body>
</html>
