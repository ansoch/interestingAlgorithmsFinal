<template>
    <div class="app">
        <div class="canvasPanel"><div class="panel"><CanvasPoints @canvasCreated="onCanvasCreated" @array-sent="onArraySent" ref="childCanvas"> </CanvasPoints></div></div>
        <div class="panel">
        <div class="buttons">
        <label>Число поколений</label> <input type = "number" v-model="gens" min = "10000" max = "1000000"/>
        <div><button @click="genAlg(this.citiesCords)">Сгенерировать путь</button></div>
        <div v-if="this.works === true"><div>Алгоритм работает</div></div>
        </div>
        </div>
    </div>    
</template>
  
  <script>
import CanvasPoints from './CanvasPoints';
  //import * as GenClasses from './GeneticClasses.js';
  export default {

    name: "GeneticAlgorithm",
    components: {
        CanvasPoints
    },
    data() {
      return {
        canvas: null,
        ctx: null,
        gens: 0,
        citiesCords: [],
        works: false
      };
    },
    methods: {
        onArraySent(array) {
            this.citiesCords = array;
        },
        onCanvasCreated(canvas) {
            this.canvas = canvas;
            this.ctx = canvas.getContext('2d');
        },
        genAlg(citiesCord){
            const self = this;
            function getRandomInt(max) {
                return Math.floor(Math.random() * max);
            }
            function getDist(cities){
                let distCities = [];

                let dist = 0;
                for (let i = 0; i < cities.length; i++){
                    distCities.push([])
                    for (let j = 0; j < cities.length; j++){
                        dist = Math.round(Math.sqrt(
                            Math.pow(cities[i][0] - cities[j][0], 2) + Math.pow(cities[i][1] - cities[j][1], 2)
                        ));
                        distCities[i].push(dist);
                    }
                }
                return distCities;
            }   
            function crossover(parent1, parent2, mutChance, distCities){
                let child1 = new Tour;
                let child2 = new Tour;
                let a = getRandomInt(parent1.tour.length);
                for (let i = 0; i < a; i++){
                    child1.tour.push(parent1.tour[i]);
                    child2.tour.push(parent2.tour[i]);
                }
                let i = 0
                while (child1.tour.length < parent2.tour.length - 1){
                    if (!child1.tour.includes(parent2.tour[i])) child1.tour.push(parent2.tour[i]);
                    i++;
                }
                i = 0;
                while (child2.tour.length < parent1.tour.length - 1){
                    if (!child2.tour.includes(parent1.tour[i])) child2.tour.push(parent1.tour[i]);
                    i++
                }
                child1.tour.push(child1.tour[0]);
                child2.tour.push(child2.tour[0]);
                child1.calculateDist(distCities);
                child2.calculateDist(distCities);
                if (getRandomInt(99) < mutChance) child1.mutate(distCities);
                if (getRandomInt(99) < mutChance) child2.mutate(distCities);
                if (child1.distance < child2.distance) return child1;
                else return child2;
            }
            function restore(cities){
				for (let i = 0; i < cities.length; i++) {
					self.ctx.beginPath();
					self.ctx.arc(citiesCord[i][0], citiesCord[i][1], 15, 0, 2 * Math.PI);
					self.ctx.fill();
				}
			}
            // eslint-disable-next-line no-unused-vars
            class City{
                constructor(num, closeCities = []) {
                    this.num = num;
                    this.closeCities = closeCities;
                }
                findCloseCities(numCities, numCloseCities, distCities){
                    for (let i = 0; this.closeCities.length < numCloseCities; i++){
                        if (i !== this.num) this.closeCities.push(i);
                    }
                    this.closeCities.sort((a, b) => {
                        return(distCities[this.num][a] - distCities[this.num][b])
                    })

                    for (let i = numCloseCities; i < numCities; i++) {
                        if (i === this.num) continue;
                        for (let j = this.closeCities.length - 1; j >= 0; j--) {
                            if (distCities[this.num][i] < distCities[this.num][this.closeCities[j]]) {
                                this.closeCities[j] = i;
                                this.closeCities.sort((a, b) => {
                                    return (distCities[this.num][a] - distCities[this.num][b])
                                })
                                break;
                            }
                        }
                    }
                }
            }
            // eslint-disable-next-line no-unused-vars
            class Tour{
                constructor(tour = [], distance = 0) {
                    this.tour = tour;
                    this.distance = distance;
                }

                generateTour(numCities, numCloseCities, distCities, cities, chanceUseCloseCity){
                    let curCity = cities[getRandomInt(numCities)];
                    this.tour.push(curCity.num);
                    let nextCity = cities[this.tour[0]];
                    for (let i = 0; i < numCities; i++){
                        //console.log(this.tour);
                        nextCity = cities[this.tour[0]];
                        if (getRandomInt(99) <= chanceUseCloseCity){
                            for (let j = 0; j < numCloseCities; j++){
                                if (!(this.tour.includes(curCity.closeCities[j]))) nextCity = cities[curCity.closeCities[j]];
                            }
                        }
                        if (nextCity === cities[this.tour[0]])for (let j = 0; j < numCities; j++){
                            if(!(this.tour.includes(j))) nextCity = cities[j];
                        }
                        this.distance += distCities[curCity.num][nextCity.num];
                        curCity = nextCity;
                        this.tour.push(curCity.num);
                    }

                }
                calculateDist(distCities){
                    this.distance = 0;
                    for (let i = 1; i < this.tour.length; i++) this.distance += distCities[this.tour[i-1]][this.tour[i]];
                }
                mutate(distCities){
                    let a = 1+getRandomInt(this.tour.length-2);
                    let b = 1+getRandomInt(this.tour.length-2);
                    if (a > b) {let t = a; a = b; b = t;}

                    let mutating = [];
                    for (let i = a; i < b; i++) mutating[i-a] = this.tour[i];
                    mutating.reverse();
                    for (let i = a; i < b; i++) this.tour[i] = mutating[i-a];
                    this.calculateDist(distCities);
                }
            }
            function drawTour(citiesCoords, tour){
                self.$refs.childCanvas.clearPoints();
                restore(citiesCoords);
                self.ctx.beginPath();
                self.ctx.moveTo(citiesCoords[tour.tour[0]][0], citiesCoords[tour.tour[0]][1]);
                for(let i = 1; i < tour.tour.length; i++){
                    self.ctx.lineTo(citiesCoords[tour.tour[i]][0], citiesCoords[tour.tour[i]][1])
                }
                self.ctx.stroke();
            }
            let numCities = citiesCord.length;
            let distCities = getDist(citiesCord);

            //
            let popSize = 100;
            let wrGroupSize = 5;
            let mutChance = 3;
            let maxGen = self.gens;
            let numCloseCities = (numCities > 5) ? 5 : numCities - 1;
            let chanceUseCloseCity = 90;
            //

            let cities = [];
            for (let i = 0; i < numCities; i++){
                let city = new City(i);
                city.findCloseCities(numCities, numCloseCities, distCities);
                cities.push(city);
            }
            let population = [];
            let bestDistance = 99999999;
            for (let i = 0; i < popSize; i++){
                let tour = new Tour;
                tour.generateTour(numCities, numCloseCities, distCities, cities, chanceUseCloseCity);
                population.push(tour);
            }
            function drawWithDelay(citiesCord, population, i) {
                setTimeout(() => {
                    drawTour(citiesCord, population[i]);
                }, 100 * i);
            }

            this.works = true;
            for (let i = 0; i < population.length; i++){
                if(population[i].distance < bestDistance){
                    drawWithDelay(citiesCord, population, i);
                    bestDistance = population[i].distance;
                    console.log(population[i].distance);
                    
                }
            }
            this.works=false;
            
            population.sort((a, b) => {return a.distance - b.distance});
            let best = population[0];
            //drawTour(citiesCord, best);
            console.log("best:", best.distance);

            let wrGroup = [];
            for (let i = 0; i < maxGen; i++){
                while(wrGroup.length < wrGroupSize) {
                    let a = getRandomInt(popSize);
                    wrGroup.push(population[a]);
                }
                wrGroup.sort((a, b) => {return a.distance - b.distance});
                //console.log(wrGroup);
                wrGroup[wrGroupSize-1] = crossover(wrGroup[0], wrGroup[1], mutChance, distCities);
                //console.log(wrGroup);
                for (let j = 0; j < wrGroupSize; j++) population[popSize-1-j] = wrGroup[j];
                wrGroup = [];
                population.sort((a, b) => {return a.distance - b.distance});

                if(population[0].distance < best.distance) {
                    best = population[0];
                    drawTour(citiesCord, best);
                }
            }
            console.log("best way: ");
            console.log(best);
            
        }
    },
  };
  </script>

<style></style>