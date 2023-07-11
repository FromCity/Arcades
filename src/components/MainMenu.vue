<template>
    <SnakeGame v-if="snakeView"/>
    <div v=else>
        <Button label="play snake" @click="playSnake"/>
    </div>
</template>

<script>
import SnakeGame from './Snake.vue'
import Button from 'primevue/button';
import { ref } from 'vue';


export default {
  name: 'MainMenu',
  components: {
    SnakeGame,
    Button
  },
  data: () => ({
    snakeView: ref(false),
  }),
  methods: {
    playSnake(){
        this.snakeView = ref(true)
    }
  },
}
</script>