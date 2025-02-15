<script>
import { h } from 'vue'
import { RefValidator } from '@/components/propValidators'
import CircuitLayer from './circuitLayer'
import RenderCircuitLayers from './renderCircuitLayers'
import { SPLIT_RENDERING } from './consts'

const BLOCK_LENGTH = 10

export default {
  name: 'split-circuit-layers',
  components: { RenderCircuitLayers, CircuitLayer },
  props: {
    registerOrder: { type: Array, required: true },
    commandRefs: {
      type: Array,
      validator: (data) => {
        return data.reduce((acc, ref) => acc && RefValidator(ref), true)
      },
      required: true
    },
    idCommandRef: { type: Object, validator: RefValidator, required: true },
    condensedRegisters: { type: Object, required: true }
  },
  emits: ['updated'],
  data () {
    return {
      readyUpTo: 0 // Track up to which block index we have finished rendering
    }
  },
  computed: {
    nBlocks () {
      return Math.ceil(this.commandRefs.length / BLOCK_LENGTH)
    },
    blocks () {
      const blocks = []
      let i = 0
      while (i < this.nBlocks) {
        blocks[i] = this.commandRefs.slice(i * BLOCK_LENGTH, (i + 1) * BLOCK_LENGTH)
        i++
      }
      return blocks
    }
  },
  render () {
    if (this.nBlocks > 0) {
      return [...Array(this.nBlocks).keys()].map(i => {
        return i <= this.readyUpTo
          ? h(
            RenderCircuitLayers,
            {
              key: i,
              registerOrder: this.registerOrder,
              idCommandRef: this.idCommandRef,
              condensedRegisters: this.condensedRegisters,
              commandRefs: this.blocks[i],
              split: this.getSplit(i),
              onLayersRendered: () => this.handleBlockUpdate(i)
            }
          )
          : i + 1 === this.nBlocks
            ? h(CircuitLayer, null, {
              default: () => [h('div', { class: 'loading' }, ['...'])]
            })
            : ''
      })
    }
    return ''
  },
  methods: {
    handleBlockUpdate (i) {
      if (this.readyUpTo <= i) {
        this.readyUpTo = i + 1
        this.$emit('updated')
      }
    },
    refresh () {
      this.readyUpTo = 0
    },
    getSplit (i) {
      return this.nBlocks > 1
        ? (i > 0
            ? (i === this.nBlocks - 1
                ? SPLIT_RENDERING.end
                : SPLIT_RENDERING.middle)
            : SPLIT_RENDERING.start)
        : SPLIT_RENDERING.none
    }
  }
}
</script>

<style>
.circuit-layer > .loading {
  padding: 0 1em;
  height: 100%;
  transform: translate(0, calc(50% - 1em) );
}
</style>
