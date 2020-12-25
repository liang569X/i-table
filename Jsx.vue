<script>
export default {
  name: 'Jsx',
  functional: true,
  props: {
    row: {
      type: Object,
      default: () => {}
    },
    Jsx: {
      type: Function,
      required: true,
      default: () => () => {}
    }
  },
  render(h, context) {
    return context.props.Jsx(h, context.props.row)
  }
}
</script>

