<template>
  <div class="cursor-pointer"
       @click="addChild">
  <Icon
    type="plus-circle"
    v-if="field.max === 0 || field.children.length < field.max"
    hover-color="success"
  />  {{__('Add')}}
  </div>
</template>

<script>
import NestedFormIcon from "./NestedFormIcon";

export default {
  components: { NestedFormIcon },

  props: {
    field: {
      type: Object,
      required: true,
      schema: {},
      children: [],
    },
  },

  methods: {
    /**
     * Add a new child.
     */
    addChild() {
      let maxKey = 0;
      if (this.field.children && this.field.children.length) {
        maxKey =
          this.field.children[this.field.children.length - 1].key ||
          Math.max.apply(
            Math,
            this.field.children.map(({ id }) => id)
          );
      }
      if (!this.field.schema) {
        this.field.schema = {};
      }
      if (!this.field.children) {
        this.field.children = [];
      }
      this.field.schema.key = maxKey + 1;
      this.field.children.push(this.replaceIndexesInSchema(this.field));
    },

    /**
     * This replaces the "{{index}}" values of the schema to
     * their actual index.
     *
     */
    replaceIndexesInSchema(field) {
      const schema = JSON.parse(JSON.stringify(field.schema));

      schema.fields &&
        schema.fields.forEach((field) => {
          if (field.schema) {
            field.schema = this.replaceIndexesInSchema(field);
          }
          if (field.attribute) {
            field.attribute = field.attribute.replace(
              this.field.indexKey,
              this.field.children.length
            );
          }
          if (field.displayIf) {
            field.displayIf = JSON.parse(
              JSON.stringify(field.displayIf).replace(
                new RegExp(this.field.indexKey, "g"),
                this.field.children.length.toString()
              )
            );
          }
        });

      schema.heading =
        (schema.heading &&
          schema.heading.replace(
            this.field.indexKey,
            this.field.children.length + 1
          )) ||
        "";

      return schema;
    },
  },

  /**
   * On created.
   */
  created() {
    for (
      let i = (this.field.children && this.field.children.length) || 0;
      i < this.field.min;
      i++
    ) {
      this.addChild();
    }
  },
};
</script>
