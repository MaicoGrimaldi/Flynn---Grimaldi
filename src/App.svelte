<script>
  import * as d3 from "d3";
  import { onMount } from "svelte";

  let bikeData = [
    { city: "Buenos Aires",  year: 2015, count: 2400 },
    { city: "Buenos Aires",  year: 2016, count: 3300 },
    { city: "Buenos Aires",  year: 2017, count: 4200 },
    { city: "Buenos Aires",  year: 2018, count: 5400 },
    { city: "Buenos Aires",  year: 2019, count: 6300 },
    { city: "Buenos Aires",  year: 2020, count: 7100 },
    { city: "Buenos Aires",  year: 2021, count: 7700 },
    { city: "Buenos Aires",  year: 2022, count: 8700 },
    { city: "Buenos Aires",  year: 2023, count: 9200 },
    { city: "Buenos Aires",  year: 2024, count: 9800 },

    { city: "Córdoba", year: 2015, count: 2300 },
    { city: "Córdoba", year: 2016, count: 3500 },
    { city: "Córdoba", year: 2017, count: 4500 },
    { city: "Córdoba", year: 2018, count: 5600 },
    { city: "Córdoba", year: 2019, count: 5000 },
    { city: "Córdoba", year: 2020, count: 4300 },
    { city: "Córdoba", year: 2021, count: 3800 },
    { city: "Córdoba", year: 2022, count: 3200 },
    { city: "Córdoba", year: 2023, count: 2800 },
    { city: "Córdoba", year: 2024, count: 2500 },

    { city: "Rosario", year: 2015, count: 3500 },
    { city: "Rosario", year: 2016, count: 3600 },
    { city: "Rosario", year: 2017, count: 3700 },
    { city: "Rosario", year: 2018, count: 3800 },
    { city: "Rosario", year: 2019, count: 4200 },
    { city: "Rosario", year: 2020, count: 4300 },
    { city: "Rosario", year: 2022, count: 4400 },
    { city: "Rosario", year: 2022, count: 4500 },
    { city: "Rosario", year: 2023, count: 4600 },
    { city: "Rosario", year: 2024, count: 4700 }
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
    <h1 class="main-title">💥<strong> EL BOOM DEL USO DE BICICLETAS EN ARGENTINA</strong>  💥</h1>
    <h2 class="subheading">En los últimos años, el uso de bicicletas en ciudades argentinas ha crecido exponencialmente</h2>
    <p class="story-text">
      La bicicleta es mucho más que un medio de transporte: es salud, es sostenibilidad y es conexión con la ciudad. En Argentina, en los últimos años su uso ha cobrado un nuevo protagonismo. Ya sea para ir al trabajo, estudiar o simplemente disfrutar del aire libre, cada vez más personas eligen pedalear. Este proyecto explora cómo ha evolucionado esta tendencia en distintas ciudades del país y qué nos cuentan los números sobre su adopción y crecimiento.
    </p>
  </div>

  <section class="city-container">
    {#each Object.keys(groupedData) as city, i}
      <div class="city-section">
        <h3 class="city-title city-title-limited">
          {#if city === "Buenos Aires"}
          <strong>Buenos Aires</strong><br>
            <span class="city-subtext">Tendencia sostenida y creciente cada año</span>
          {:else if city === "Córdoba"}
            <strong>Córdoba</strong><br>
            <span class="city-subtext">Crecimiento acelerado desde 2018</span>
          {:else if city === "Rosario"}
            <strong>Rosario</strong><br>
            <span class="city-subtext">Estabilidad con leve incremento continuo</span>
          {/if}
        </h3>
      </div>
      <div class={i < 3 ? "city-card equal-height fade-in" : "city-card fade-in"}>
    
        <div class="bike-list">
          {#each groupedData[city] as d}
            <div class="bike-entry">
              <span class="bike-year">{d.year}</span>
              <div class="progress-bar"
                  style="width: {pct(d)}; background-color: {color(d.count)};"></div>
              <span class="bike-count">{d.count}</span>
              <span class="plant-icons">
                {#each Array(plantScale(d.perThousand)) as _, idx}
                  <span class="plant-icon">🌱</span>
                {/each}
              </span>
              <span class="per-thousand-count">{d.perThousand.toFixed(1)} ‰</span>
            </div>
          {/each}
        </div>
        <div class="card-legend-inline">
          <div class="legend-entry">
            <div class="progress-bar legend-bar"></div>
            <span class="legend-text">Barra proporcional al total anual de bicicletas</span>
          </div>
          <div class="legend-entry">
            <span class="plant-icon">🌱</span>
            <span class="legend-text">     Nivel de adopción (bicis cada 1000 hab.)</span>
          </div>
        </div>        
      </div>
    {/each}
</section>

<h2 class="deep-dive-title"><strong>Comparación de tendencias entre ciudades</strong></h2>

<section class="city-card flourish-paired fade-in">
    <div class="flourish-double">
      <!-- Primer gráfico + texto -->
      <!-- Primer gráfico + texto -->
  <div class="flourish-single">
    <div class="flourish-chart-container">
      <div class="flourish-embed flourish-chart" data-src="visualisation/22921002"></div>
    </div>
    <div class="flourish-chart-text">
      <h2 class="chart-title"> Evolución desigual entre ciudades argentinas</h2>
      <p>
        Buenos Aires muestra un crecimiento constante en la cantidad de bicicletas desde 2015, consolidando año a año su liderazgo. Córdoba, en cambio, exhibe una trayectoria opuesta: luego de un inicio prometedor, cae de forma sostenida a partir de 2018. Rosario mantiene un comportamiento más equilibrado, sin grandes saltos pero también sin retrocesos marcados.
     </p>
    </div>
  </div>

  <!-- Segundo gráfico + texto -->
  <div class="flourish-single">
    <div class="flourish-chart-container">
      <div class="flourish-embed flourish-chart" data-src="visualisation/22654728"></div>
    </div>
    <div class="flourish-chart-text">
      <h2 class="chart-title">Córdoba pierde el liderazgo, Buenos Aires toma la delantera</h2>
      <p>
        El crecimiento porcentual acumulado evidencia un cambio de posiciones entre las ciudades. Buenos Aires multiplica por cuatro su parque de bicicletas en menos de una década, mientras que Rosario crece de forma más suave pero constante. Córdoba, que había arrancado con fuerza, pierde empuje y queda muy por debajo del ritmo de sus pares.
      </p>
    </div>
  </div>

</section>

  
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

  /* INICIO */

  .intro {
    max-width: 1100px;
    margin: 0 auto 2rem;
    text-align: center;
    
  }

  .main-title {
    font-size: 2.5rem;
    font-weight: 800;
    margin-bottom: 0.5rem;
    color: #111;
  }

  .subheading { 
    font-size: 1.4rem; 
    margin: 0.5rem auto;
    font-weight: 500; 
    max-width: 1100px;
  }

  .story-text {
    max-width: 840px;    /* ajusta este valor a tu gusto */
    width: 90%;          /* opcional: que sea un % del contenedor */
    margin: 1rem auto;   /* centra horizontalmente y separa verticalmente */
  }

  /* CIUDADES */

  .city-container { display: grid; gap: 2rem; }
  .city-section { margin-bottom: 1rem; }
  .city-title { font-size: 2rem; font-weight: 600; text-align: center; user-select: none;}
  .city-title-limited {max-width: 1000px; margin: 0 auto; line-height: 1.3;}
  .city-title-limited strong {
    font-weight: 800;
    color: #233;
  }

  .city-subtext {
    font-weight: 700;
    color: #666; /* un gris más suave */
    font-size: 1.5rem;
  }
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

  /* AÑOS Y DATA */

  .bike-list { display: flex; flex-direction: column; gap: 1.6rem; }

  .bike-entry {
    display: grid;
    grid-template-columns: 60px 1fr auto;
    grid-template-rows: auto auto;
    align-items: center;
    gap: 0.6rem 1rem;  
    margin-bottom: 1.6rem; 
  }

  .bike-year {
    grid-row: 1 / span 2;
    align-self: center;
    font-size: 0.85rem;  
    font-weight: 600;
    text-align: center;
  }

  /* Primera fila: bicicletas */

  .progress-bar {
    height: 32px;
    border-radius: 16px;
    overflow: hidden;
    background-color: currentColor; /* tu color de barra */

    -webkit-mask-image: url('./images/bici.svg');
    -webkit-mask-size: auto 100%;
    -webkit-mask-repeat: space;
    -webkit-mask-position: center;

    mask-image: url('./images/bici.svg');
    mask-size: auto 100%;
    mask-repeat: space;
    mask-position: center;
  }

  .bike-count {
    grid-row: 1;
    grid-column: 3;
    font-size: 1.6rem;
    font-weight: 600;
  }

  /* Segunda fila: plantitas */
  .plant-icons {
    grid-row: 2;
    grid-column: 2;
    display: flex;
    gap: 0.2rem;
  }
  .per-thousand-count {
    grid-row: 2;
    grid-column: 3;
    font-size: 0.95rem;
    font-weight: 500;
    color: #388E3C;
  }

    /* LEYENDA */
    .card-legend-inline {
      display: flex;
      justify-content: center;
      align-items: center;
      flex-wrap: wrap;
      gap: 2.5rem;
      margin: 2rem auto 0 auto;
      padding: 0.8rem 1.2rem;
      border-radius: 12px;
      background-color: #f9f9f9;
      max-width: 900px;
      flex-wrap: nowrap;
    }

    .legend-entry {
      display: flex;
      align-items: center;
      gap: 0.6rem;
    }

    .legend-bar {
      width: 32px; /* más chico que antes para que entre 1 bici */
      height: 32px;
      background-color: #444; /* color fijo y oscuro */
      -webkit-mask-image: url('./images/bici.svg');
      -webkit-mask-size: contain;
      -webkit-mask-repeat: no-repeat;
      -webkit-mask-position: center;

      mask-image: url('./images/bici.svg');
      mask-size: contain;
      mask-repeat: no-repeat;
      mask-position: center;
      border-radius: 12px;
    }


    .legend-text {
      font-size: 0.9rem;
      font-weight: 500;
      color: #333;
    }

  /* GRAFICOS */
  
  .deep-dive-title {
    font-size: 1.8rem;
    font-weight: 600;
    color: #444; 
    text-align: center;
    margin: 0;
    padding-bottom: 4rem;
    margin-top: 3rem;
    user-select: none;
  }

  .flourish-double {
    display: flex;
    flex-direction: column;
    gap: 5rem; /* espacio entre los dos bloques */
  }

  .flourish-single {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    gap: 2rem;
  }


  .chart-title {
    font-size: 2.2rem;
    font-weight: 800;
    color: #2c3e50; /* azul oscuro, sobrio */
    margin-bottom: 1rem;
    border-left: 6px solid #000000;
    padding-left: 1rem;
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

    padding: rem
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

  /* FOOTER */

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

  /* ANIMACIONES */
  
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
