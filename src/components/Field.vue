<template>
    <div>
        <font-awesome-icon :icon="icon" />
    </div>
</template>
<script>
export default {
  name: 'Field',
  props: ['value'],
  computed: {
      icon: function() {
          if(this.$props.value==1)
            return "angry"
          else if (this.$props.value==10)
            return "smile-beam"
          else 
            return "circle"
      }
  }
}
</script>