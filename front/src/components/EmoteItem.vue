<script>
import emojis from "@/../public/config/emoji.json"

export default {
  props: {
    id: {
      type: [String],
      default: ''
    },
    size: {
      type: [String, Number],
      default: '24'
    },
    isSpan: {
      type: Boolean,
      default: true
    },
    isDisabledTooltip: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      emojiItemData: null
    }
  },
  created() {
    this.getInquireEmojiItem(this.id);
  },
  watch: {
    id: {
      handler(val) {
        this.getInquireEmojiItem(val);
      }
    }
  },
  methods: {
    /**
     * 查询对应表情id
     * {type}|{id}
     * @param idRaw
     * @returns {{}}
     */
    getInquireEmojiItem(idRaw) {
      let type = idRaw.split('|')[0],
          id = idRaw.split('|')[1],
          result = {
            isCustom: false,
          };

      const typeList = this.emojis.child.findLast(i => i.name === type);
      if (typeList && typeList.child)
        for (let index = 0; index <= typeList.child.length; index++) {
          let i = typeList.child[index];
          if (i && i.name === id) {
            result = {
              isCustom: false,
              imageUrl: typeList.imageUrl,
              ...i,
            };
            break;
          }
        }

      if (type === 'custom-network-https' || type === 'custom-network') {
        result = {
          isCustom: true,
          name: type,
          imageUrl: `https://${id}`
        }
      }
      if (type === 'custom-network-http') {
        result = {
          isCustom: true,
          name: type,
          imageUrl: `http://${id}`
        }
      }

      this.emojiItemData = result;
      return result;
    }
  },
  computed: {
    emojis: () => emojis
  }
}
</script>

<template>
  <Tooltip v-if="emojiItemData && emojiItemData.config" :disabled="isDisabledTooltip"
           placement="top" transfer>
    <template v-slot:content>
      <div align="center">
        <br>
        <template
            v-if="emojiItemData && emojiItemData.config && (emojiItemData.config.type === 'gif' || emojiItemData.config.type === 'png')">
          <img class="emote"
               :alt="emojiItemData.name"
               :src="emojiItemData.imageUrl"
               :class="[
                 isSpan ? 'emote-none-padding': ''
              ]"
               :style="`width: ${size * 3}px ;height: ${size * 3}px`"
               v-if="emojiItemData"/>
        </template>
        <template v-else-if="emojiItemData && emojiItemData.config && emojiItemData.config.type === 'spriteDiagram'">
          <span class="emote"
                :class="[
                   isSpan ? 'emote-none-padding': ''
                ]"
                :style="`width: ${size * 3}px;height: ${size * 3}px;background-image: url(${emojiItemData.imageUrl});background-size: ${emojiItemData.config.size};background-position: ${emojiItemData.config.position}`"
                v-if="emojiItemData">
            <img src="data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7"
                 :alt="emojiItemData.name"/>
          </span>
        </template>

        <p class="emote-name">{{emojiItemData.name}}</p>
      </div>
    </template>
    <template
        v-if="emojiItemData && emojiItemData.config && (emojiItemData.config.type === 'gif' || emojiItemData.config.type === 'png')">
      <img class="emote"
           :alt="emojiItemData.name"
           :src="emojiItemData.imageUrl"
           :class="[
                 isSpan ? 'emote-none-padding': ''
              ]"
           :style="`width: auto ;height: ${size}px`"
           v-if="emojiItemData"/>
    </template>
    <template v-else-if="emojiItemData && emojiItemData.config && emojiItemData.config.type === 'spriteDiagram'">
        <span class="emote"
              :class="[
                 isSpan ? 'emote-none-padding': ''
              ]"
              :style="`width: ${size}px;height: ${size}px;background-image: url(${emojiItemData.imageUrl});background-size: ${emojiItemData.config.size};background-position: ${emojiItemData.config.position}`"
              v-if="emojiItemData">
          <img src="data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7"
               :alt="emojiItemData.name"/>
        </span>
    </template>
  </Tooltip>
  <span v-else>
    <template v-if="emojiItemData && emojiItemData.isCustom">
      <img class="emote"
           :alt="emojiItemData.name"
           :src="emojiItemData.imageUrl"
           :class="[
                 isSpan ? 'emote-none-padding': ''
              ]"
           :style="`width: auto ;height: ${size}px`"
           v-if="emojiItemData"/>
    </template>
    <template v-else>
      [{{ id }}]
    </template>
  </span>
</template>

<style scoped lang="less">
span.emote-none-padding,
img.emote-none-padding {
  margin: -6px 1px;
}

span.emote,
img.emote {
  display: inline-flex !important;
  position: relative;
  cursor: pointer;

  .emote-name {
    opacity: .8;
    margin-top: 4px;
    font-size: 12px;
  }

  img {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    width: 100%;
    height: 100%;
  }
}
</style>
