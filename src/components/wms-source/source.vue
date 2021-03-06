<script>
  import TileWMSSource from 'ol/source/tilewms'
  import { omit } from 'lodash/fp'
  import { WMS_VERSION, tileSource, assert } from '../../core'

  const props = {
    extParams: Object, // Additional WMS Request params
    gutter: Number,
    hidpi: Boolean,
    layers: {
      type: String,
      required: true,
    },
    serverType: String,
    styles: String, // WMS Request styles
    version: {
      type: String,
      default: WMS_VERSION,
    },
    url: {
      type: String,
      required: true,
    },
  }

  const upperCase = x => x.toUpperCase()
  const keysToUpperCase = x => Object.keys(x).map(upperCase)
  const cleanExtParams = params => omit(['LAYERS', 'VERSION', 'STYLES'], keysToUpperCase(params))

  const methods = {
    /**
     * @returns {ol.source.TileWMS}
     * @protected
     */
    createSource () {
      return new TileWMSSource({
        attributions: this.attributions,
        cacheSize: this.cacheSize,
        params: {
          ...cleanExtParams(this.extParams),
          LAYERS: this.layers,
          STYLES: this.styles,
          VERSION: this.version,
        },
        crossOrigin: this.crossOrigin,
        gutter: this.gutter,
        hidpi: this.hidpi,
        logo: this.logo,
        tileGrid: this._tileGrid,
        projection: this.projection,
        reprojectionErrorThreshold: this.reprojectionErrorThreshold,
        serverType: this.serverType,
        wrapX: this.wrapX,
        url: this.urlTmpl,
      })
    },
    /**
     * @param {number[]} coordinate
     * @param {number} [resolution]
     * @param {string} [projection]
     * @param {Object} [params] GetFeatureInfo params. `info_format` at least should be provided.
     *                          If `query_layers` is not provided then the layers specified in the `layers` prop will be used.
     *                          `version` should not be specified here (value from `version` prop will be used).
     * @return {string|undefined}
     */
    getFeatureInfoUrl (
      coordinate,
      resolution,
      projection,
      params = {}
    ) {
      assert.hasView(this)
      assert.hasSource(this)

      resolution || (resolution = this.$view.getResolution())
      projection || (projection = this.projection)

      return this.$source.getFeatureInfoUrl(
        coordinate,
        resolution,
        projection,
        cleanExtParams(params)
      )
    },
  }

  const watch = {
    layers (LAYERS) {
      this.$source && this.$source.updateParams({ LAYERS })
    },
    version (VERSION) {
      this.$source && this.$source.updateParams({ VERSION })
    },
    styles (STYLES) {
      this.$source && this.$source.updateParams({ STYLES })
    },
    extParams (value) {
      this.$source && this.$source.updateParams(cleanExtParams(value))
    },
  }

  export default {
    name: 'vl-source-wms',
    mixins: [tileSource],
    props,
    methods,
    watch,
  }
</script>
