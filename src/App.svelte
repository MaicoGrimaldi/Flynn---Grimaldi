<script>
  import * as d3 from "d3";
  import { onMount } from "svelte";

  let bikeData = [
    { city: "Buenos Aires",  year: 2015, count: 4200 },
    { city: "Buenos Aires",  year: 2016, count: 6500 },
    { city: "Buenos Aires",  year: 2017, count: 7000 },
    { city: "Buenos Aires",  year: 2018, count: 7500 },
    { city: "Buenos Aires",  year: 2019, count: 8000 },
    { city: "Buenos Aires",  year: 2020, count: 8500 },
    { city: "Buenos Aires",  year: 2021, count: 9000 },
    { city: "Buenos Aires",  year: 2022, count: 9500 },

    { city: "Córdoba", year: 2015, count: 2000 },
    { city: "Córdoba", year: 2016, count: 2500 },
    { city: "Córdoba", year: 2017, count: 3000 },
    { city: "Córdoba", year: 2018, count: 3500 },
    { city: "Córdoba", year: 2019, count: 4500 },
    { city: "Córdoba", year: 2020, count: 5000 },
    { city: "Córdoba", year: 2021, count: 6000 },
    { city: "Córdoba", year: 2022, count: 7000 },

    { city: "Rosario", year: 2015, count: 3500 },
    { city: "Rosario", year: 2016, count: 3600 },
    { city: "Rosario", year: 2017, count: 3700 },
    { city: "Rosario", year: 2018, count: 3800 },
    { city: "Rosario", year: 2019, count: 4200 },
    { city: "Rosario", year: 2020, count: 4300 },
    { city: "Rosario", year: 2021, count: 4400 },
    { city: "Rosario", year: 2022, count: 4500 }
  ];

  const populations = {
    "Buenos Aires": 3000000,
    "Córdoba": 600000,
    "Rosario": 400000
  };

  bikeData.forEach(d => {
    const pop = populations[d.city];
    d.perThousand = pop ? (d.count / pop) * 1000 : 0;
  });

  const [minCount, maxCount] = d3.extent(bikeData, d => d.count);
  const color = d3.scaleLinear()
    .domain([minCount, maxCount])
    .range(["#FF5733", "#4CAF50"]);
  const pct = d => (d.count / maxCount * 100) + "%";

  const [, maxPT] = d3.extent(bikeData, d => d.perThousand);
  const plantScale = d3.scaleQuantize([0, maxPT], [1,2,3,4,5,6,7,8,9,10]);

  const citiesOrder = ["Buenos Aires", "Córdoba", "Rosario"];
  bikeData.sort((a, b) =>
    citiesOrder.indexOf(a.city) - citiesOrder.indexOf(b.city) ||
    a.year - b.year
  );
  let groupedData = {};
  bikeData.forEach(entry => (groupedData[entry.city] ??= []).push(entry));

  onMount(() => {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          entry.target.classList.add("fade-in");
          observer.unobserve(entry.target);
        }
      });
    });

    document.querySelectorAll(".city-card, .flourish-paired").forEach(el => {
      observer.observe(el);
    });
  });
</script>

<main class="container">
  <div class="intro">
    <h1 class="main-title">💥 El boom del uso de bicicletas en Argentina 💥</h1>
    <h2 class="subheading">En los últimos años, el uso de bicicletas en ciudades argentinas ha crecido exponencialmente 📈</h2>
    <p class="story-text">
      Desde 2015 hasta 2022, el número de bicicletas en circulación ha mostrado tendencias interesantes:
      BS AS crece de forma constante, Córdoba acelera a partir de 2018 y Rosario mantiene estabilidad progresiva.
      Además, analizamos cómo evolucionó el uso de bicicletas en proporción a la población de cada ciudad.
    </p>
  </div>

  <div class="legend">
    🌱 Nivel de adopción (bicis por cada 1000 habitantes)
  </div>

  <section class="city-container">
    {#each Object.keys(groupedData) as city, i}
      <div class="city-section">
        <h3 class="city-title">{city}</h3>
      </div>
      <div class={i < 3 ? "city-card equal-height fade-in" : "city-card fade-in"}>
        <div class="bike-list">
          {#each groupedData[city] as d}
            <div class="bike-row">
              <span class="bike-year">{d.year}</span>
              <div class="progress-bar" style="width: {pct(d)}; background-color: {color(d.count)}; -webkit-mask: url('./images/bici.svg') repeat-x center/auto 100%; mask: url('./images/bici.svg') repeat-x center/auto 100%;"></div>
              <span class="bike-count">{d.count}</span>
            </div>
            <div class="bike-row plant-row">
              <span class="bike-year"></span>
              <span class="plant-icons">
                {#each Array(plantScale(d.perThousand)) as _, idx}
                  <span class="plant-icon">🌱</span>
                {/each}
              </span>
              <span class="per-thousand-count">{d.perThousand.toFixed(1)} ‰</span>
            </div>
          {/each}
        </div>
      </div>
    {/each}
  </section>

  <h2 class="deep-dive-title">Analizando en profundidad…</h2>

  <section class="city-card flourish-paired fade-in">
    <div class="flourish-chart-container">
      <div class="flourish-embed flourish-chart" data-src="visualisation/22921002"></div>
    </div>
    <div class="flourish-chart-text">
      <h2>Contribución acumulada por ciudad (2015–2022)</h2>
      <p>
        Este gráfico de áreas muestra cómo se distribuye la cantidad total de bicicletas entre las tres ciudades a lo largo del tiempo. 
        Permite visualizar el crecimiento conjunto y, al mismo tiempo, identificar el peso relativo de cada ciudad dentro del total acumulado.
      </p>
    </div>
  </section>

  <section class="city-card flourish-paired fade-in">
    <div class="flourish-chart-container">
      <div class="flourish-embed flourish-chart" data-src="visualisation/22654728"></div>
    </div>
    <div class="flourish-chart-text">
      <h2>Aumento porcentual del uso de bicicletas (2015–2022)</h2>
      <p>
        Este gráfico muestra el crecimiento porcentual acumulado en cada ciudad desde 2015. 
        Permite comparar la intensidad del aumento relativo, independientemente de la cantidad total de bicicletas, 
        resaltando qué ciudades adoptaron más rápidamente su uso.
      </p>
    </div>
  </section>
  <script src="https://public.flourish.studio/resources/embed.js"></script>
  
  <footer class="city-card footer fade-in">
    <p><strong>Parcial – Visualización de Datos</strong></p>
    <p>Universidad Torcuato Di Tella · Abril 2025</p>
    <div class="footer-authors">
      <div class="author">
        <h4>Maico Grimaldi</h4>
        <div class="social">
          <a href="https://instagram.com/maico_grimaldi" target="_blank"><img src="https://cdn.jsdelivr.net/npm/simple-icons@v9/icons/instagram.svg" width="28" /></a>
          <a href="https://linkedin.com/in/maico-grimaldi" target="_blank"><img src="https://cdn.jsdelivr.net/npm/simple-icons@v9/icons/linkedin.svg" width="28" /></a>
        </div>
      </div>
      <div class="author">
        <h4>Mateo Flynn</h4>
        <div class="social">
          <a href="https://instagram.com/mateoflynn__" target="_blank"><img src="https://cdn.jsdelivr.net/npm/simple-icons@v9/icons/instagram.svg" width="28" /></a>
          <a href="https://linkedin.com/in/mateo-flynn-2a5b40238/" target="_blank"><img src="https://cdn.jsdelivr.net/npm/simple-icons@v9/icons/linkedin.svg" width="28" /></a>
        </div>
      </div>
    </div>
  </footer>
</main>

<style>

  .intro {
    max-width: 1000px;
    margin: 0 auto 2rem;
    text-align: center;
    
  }

  .main-title {
    font-size: 2.5rem;
    font-weight: 700;
    margin-bottom: 0.5rem;
    color: #333;
  }

  .subheading { font-size: 1.4rem; margin: 0.5rem 0; font-weight: 500; }
  .story-text { font-size: 1rem; line-height: 1.6; }

  .city-container { display: grid; gap: 2rem; }
  .city-section { margin-bottom: 1rem; }
  .city-title { font-size: 2.4rem; font-weight: 600; text-align: center; user-select: none;}

  .city-card {
    background: #fff;
    padding: 2rem;
    border-radius: 15px;
    box-shadow: 0 6px 20px rgba(0,0,0,0.15);
    width: 92%;
    max-width: 1000px;
    margin: 0 auto 3rem;
  }
  
  .city-card.equal-height {
    min-height: 450px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
  }

  .legend {
    text-align: center;
    margin: 1rem 0;
    font-weight: 600;
    color: #388E3C;
  }

  .bike-list { display: flex; flex-direction: column; gap: 1.6rem; }

  .bike-row {
    display: grid;
    grid-template-columns: 60px 1fr auto;
    gap: 1rem;
    align-items: center
  }

  .bike-row .bike-year {
    align-self: flex-start;
    margin-top: 0.6rem;
    font-size: 1.4rem;
    font-weight: 600;
    color: #333;
    text-align: right;
  }

  .progress-bar {
    height: 32px;
    border-radius: 16px;
    overflow: hidden;
  }

  .bike-count {
    font-size: 1rem;
    font-weight: 600;
    white-space: nowrap;
  }

  .plant-row {
    display: grid;
    grid-template-columns: 60px 1fr auto;
    align-items: center;
    gap: 1rem;
  }

  .plant-icons { display: flex; gap: 0.2rem; }
  .plant-icon { font-size: 1.2rem; user-select: none;}
  .per-thousand-count {
    font-size: 0.95rem;
    font-weight: 500;
    color: #388E3C;
  }

  .deep-dive-title {
    font-size: 1.8rem;
    font-weight: 600;
    color: #444; 
    text-align: center;
    margin: 0;
    padding-bottom: 0.6rem;
    margin-top: 3rem;
    user-select: none;
  }

  .flourish-paired {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    gap: 2rem;
    padding: 2rem;
    background: #ffffff;

    border-radius: 15px;
    box-shadow: 0 6px 20px rgba(0,0,0,0.15);
    width: 92%;
    max-width: 1000px;
    margin: 0 auto 3rem;
  }

  .flourish-chart-container {
    flex: 1 1 60%;
    display: flex;
    justify-content: center;
  }

  .flourish-embed.flourish-chart {
    width: 100%;
    min-width: 300px;
    min-height: 300px;
  }

  .flourish-chart-text {
    flex: 1 1 35%;
    text-align: left;
  }

  .flourish-chart-text h2 {
    font-size: 1.4rem;
    margin-bottom: 1rem;
  }

  .flourish-chart-text p {
    font-size: 1.05rem;
    line-height: 1.6;
  }

  .footer {
    background: #ffffff;
    padding: 1.2rem 2rem 2rem 2rem; 
    border-radius: 15px;
    box-shadow: 0 6px 20px rgba(0,0,0,0.15);
    text-align: center;
    width: 92%;
    max-width: 1000px;
    margin: 0 auto 3rem;
    color: #333;
  }

  .footer-authors {
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    gap: 4rem;
    margin-top: 1rem;
  }
  .author {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .author h4 {
    font-size: 1.1rem;
    margin-bottom: 0.5rem;
  }
  .social {
    display: flex;
    gap: 0.8rem;
  }
  .social a img {
    width: 28px;
    height: 28px;
    transition: transform .2s ease;
  }
  .social a:hover img {
    transform: scale(1.1);
  }

  .city-card,
  .flourish-paired {
    scroll-margin-top: 6rem;
    transition: all 0.6s ease-in-out;
  }

  @keyframes fadeInUp {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .fade-in {
    opacity: 0;
    animation: fadeInUp 0.8s ease-out forwards;
  }

  .city-card:hover,
  .flourish-paired:hover {
    transform: translateY(-6px);
    transition: transform 0.3s ease, box-shadow 0.3s ease;
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.18);
  }

  @keyframes bounceIn {
    0% {
      transform: scale(0.5);
      opacity: 0;
    }
    60% {
      transform: scale(1.2);
      opacity: 1;
    }
    100% {
      transform: scale(1);
    }
  }

  .plant-icon {
    font-size: 1.2rem;
    user-select: none;
    animation: bounceIn 0.4s ease-in-out;
  }

</style>
