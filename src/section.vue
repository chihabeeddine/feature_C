<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div>
        <!-- wwManager:start -->
        <wwSectionEditMenu :sectionCtrl="sectionCtrl" :options="openOptions"></wwSectionEditMenu>
        <!-- wwManager:end -->
        <wwObject class="background" :ww-object="section.data.bg" ww-category="background"></wwObject>

        <div class="container hidden-mobile">
            <div class="container-center">
                <div class="thumbnail-container" v-for="feature in section.data.features" :key="feature.uniqueId" :style="columnWidth">
                    <!-- wwManager:start -->
                    <wwContextMenu
                        tag="div"
                        class="contextmenu"
                        v-if="editMode"
                        @ww-add-before="addFeature(index, 'before')"
                        @ww-add-after="addFeature(index, 'after')"
                        @ww-remove="removeFeature(index)"
                    >
                        <div class="wwi wwi-config"></div>
                    </wwContextMenu>
                    <!-- wwManager:end -->
                    <wwObject class="background" :ww-object="feature.background" ww-category="background"></wwObject>

                    <wwLayoutColumn tag="div" ww-default="ww-image" :ww-list="feature.contents" class="content" @ww-add="add(feature.contents, $event)" @ww-remove="remove(feature.contents, $event)">
                        <wwObject tag="div" v-for="content in feature.contents" :key="content.uniqueId" :ww-object="content"></wwObject>
                    </wwLayoutColumn>
                </div>
            </div>
        </div>

        <v-touch ref="swiper" @swipeleft="nextSlide()" @swiperight="prevSlide()" class="container mobile-wrapper">
            <div class="container-center" :style="[mobileStyle, mobileTransition]">
                <div class="thumbnail-container" v-for="feature in section.data.features" :key="feature.uniqueId" :style="cardWidth">
                    <!-- wwManager:start -->
                    <wwContextMenu
                        tag="div"
                        class="contextmenu"
                        v-if="editMode"
                        @ww-add-before="addFeature(index, 'before')"
                        @ww-add-after="addFeature(index, 'after')"
                        @ww-remove="removeFeature(index)"
                        @ww-options="customizeColor()"
                    >
                        <div class="wwi wwi-config"></div>
                    </wwContextMenu>
                    <!-- wwManager:end -->
                    <wwObject class="background" :ww-object="feature.background" ww-category="background"></wwObject>

                    <wwLayoutColumn tag="div" ww-default="ww-image" :ww-list="feature.contents" class="content" @ww-add="add(feature.contents, $event)" @ww-remove="remove(feature.contents, $event)">
                        <wwObject tag="div" v-for="content in feature.contents" :key="content.uniqueId" :ww-object="content"></wwObject>
                    </wwLayoutColumn>
                </div>
            </div>
            <div class="content-dots-wrapper">
                <li
                    v-for="(dot, index) in section.data.features"
                    class="content-dot"
                    :style="{'background': ((sliderPosition == index) ? section.data.dotColor : ''), 'border-color': section.data.dotColor}"
                    :key="dot.uniqueId"
                >
                    <div class="dot" @click="switchToIndex(sliderPosition, index)"></div>
                </li>
            </div>
        </v-touch>
    </div>
</template>

<!-- This is your Javascript -->
<!-- ✨ Here comes the magic ✨ -->
<script>

const VueTouch = require('vue-touch')

Vue.use(VueTouch, { name: 'v-touch' })

export default {
    name: "__COMPONENT_NAME__",
    props: {
        // The section controller object is passed to you.
        sectionCtrl: Object
    },
    data() {
        return {
            columnWidth: { 'width': 'calc(33.33% - 30px)' }, //TODO:
            sliderPosition: 0
        }
    },
    computed: {
        //Get the section object here !
        // It contains all the info and data about the section
        // Use it has you like !
        section() {
            return this.sectionCtrl.get();
        },
        editMode() {
            return this.sectionCtrl.getEditMode() == 'CONTENT'
        },

        featuresLength() {
            return this.section.data.features.length
        },
        mobileStyle() {
            return { 'width': 'calc(' + this.featuresLength + ' * 100%)' }
        },
        mobileTransition() {
            return { 'transform': 'translate(calc(' + (- this.sliderPosition * (100 / this.featuresLength)) + '% + ' + this.sliderPosition * 32 + 'px ), 0)' }
        },
        cardWidth() {
            return { 'width': 'calc(' + (100 / this.featuresLength) + ' - 50px)' }
        }
    },

    /* add  margin-left 25px, width calc( 100/ nmber -50px) margin-left 15px */
    created() {

        let needUpdate = false
        //Initialize section data
        this.section.data = this.section.data || {};
        this.section.data.dotColor = this.section.data.dotColor || "#9013FE"

        if (!this.section.data.bg) {
            this.section.data.bg = wwLib.wwObject.getDefault({
                type: 'ww-color'
            });
            needUpdate = true
        }

        if (!this.section.data.features) {
            this.section.data.features = []
            needUpdate = true
        }
        if (!this.section.data.thumbnailsPerLine) {
            this.section.data.thumbnailsPerLine = 4;
            needUpdate = true;
        }

        if (_.isEmpty(this.section.data.features)) {
            this.section.data.features.push(
                {
                    background: wwLib.wwObject.getDefault({ type: 'ww-color', data: { color: 'white' } }),
                    contents: []
                },
                {
                    background: wwLib.wwObject.getDefault({ type: 'ww-color', data: { color: 'white' } }),
                    contents: []
                },
                {
                    background: wwLib.wwObject.getDefault({ type: 'ww-color', data: { color: 'white' } }),
                    contents: []
                },

            )
            needUpdate = true;
        }


        if (needUpdate) {
            this.sectionCtrl.update(this.section);
        }


    },
    mounted() {
        if (this.editMode)
            this.$refs.swiper.disable('swipe')
    },
    methods: {
        nextSlide() {
            try {
                this.sliderPosition++
                if (this.sliderPosition > this.featuresLength - 1)
                    this.sliderPosition = 0
                this.$forceUpdate()
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', err);
            }
        },

        prevSlide() {
            try {
                this.sliderPosition--
                if (this.sliderPosition < 0)
                    this.sliderPosition = this.featuresLength - 1
                this.$forceUpdate()
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', err);
            }

        },
        switchToIndex(index, position) {
            if (position < this.section.data.features.length && index != position) {
                this.sliderPosition = position
            }
        },

        /* wwManager:start */
        add(list, options) {
            list.splice(options.index, 0, options.wwObject);
            this.sectionCtrl.update(this.section);
        },


        /* add a new section to the slider */
        addFeature(_index, where) {
            try {
                const up = (where == 'after') ? parseInt(1) : 0;
                const index = _index + up
                const newCard = {
                    background: wwLib.wwObject.getDefault({ type: 'ww-color', data: { color: 'white' } }),
                    contents: []
                }

                this.section.data.features.splice(index, 0, newCard);
                this.sectionCtrl.update(this.section);
                this.columnWidth = { 'width': "calc(" + 100 / (this.featuresLength) + "%   - 30px)" }
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', err);
            }
        },

        removeFeature(index) {

            this.section.data.features.splice(index, 1);
            if (!this.section.data.features.length) {
                this.addFeature(0, 'after');
            }
            this.sectionCtrl.update(this.section);
            this.columnWidth = { 'width': "calc(" + 100 / (this.featuresLength) + "%   - 30px)" }
        },

        /* Open a popup to change the border color */
        async customizeColor() {
            try {
                wwLib.wwObjectHover.setLock(this);

                let options = await this.edit()
                const result = await wwLib.wwPopups.open(options);
                /*=============================================m_ÔÔ_m=============================================\
                  STYLE
                \================================================================================================*/
                if (typeof (result) != 'undefined') {
                    if (typeof (result.dotsColor) != 'undefined') {
                        this.section.data.dotColor = result.dotsColor
                    }
                    this.sectionCtrl.update(this.section);
                }
                wwLib.wwObjectHover.removeLock();
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', err);
            }
        },
        /* create a popup forum */
        async edit() {
            wwLib.wwPopups.addStory('WWSLIDER_CUSTOM', {
                title: {
                    en: 'Color picker',
                    fr: 'Choisir une couleur'
                },
                type: 'wwPopupForm',
                storyData: {
                    fields: [
                        {
                            label: {
                                en: 'Navigation dots color:',
                                fr: 'Couleur des points de navigations :'
                            },
                            type: 'color',
                            key: 'dotsColor',
                            valueData: 'section.data.dotColor',
                            desc: {
                                en: 'Choose a Nav dots color:',
                                fr: 'Changer la couleur des points de navigations :'
                            }
                        },
                    ]
                },
                buttons: {

                    NEXT: {
                        text: {
                            en: 'Ok',
                            fr: 'Ok'
                        },
                        next: false
                    }
                }
            })

            let options = {
                firstPage: 'WWSLIDER_CUSTOM',
                data: {
                    section: this.section,
                }
            }
            return options
        },

        async openOptions() {
            let options = {
                firstPage: 'FEATURE_C_COLUMN_COUNT',
                data: {
                    columnPerLine: this.section.data.thumbnailsPerLine
                },
            }

            const result = await wwLib.wwPopups.open(options)

            if (result.columnPerLine) {
                this.section.data.thumbnailsPerLine = result.columnPerLine;

                this.sectionCtrl.update(this.section);

                this.setThumbnailsPerLine();
            }


            console.log(result);
        },
        /* Used on mobile version to swipe */


        remove(list, options) {
            try {
                list.splice(options.index, 1);
                this.sectionCtrl.update(this.section);
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', err);
            }

        },

        /* wwManager:end */



    }
};
</script>

<!-- This is your CSS -->
<!-- Add "scoped" attribute to limit CSS to this component only -->
<!-- Add lang="scss" or others to use computed CSS -->
<style lang='scss' scoped>
.background {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
}

.container {
    .container-center {
        display: flex;
        transition: transform 0.5s ease;
        @media (min-width: 768px) {
            justify-content: center;
            flex-wrap: wrap;
        }
        .thumbnail-container {
            width: 30%;
            margin-left: 15px;
            position: relative;
            //margin: 30px 15px;
            background-color: white;
            min-height: 50px;
            box-shadow: 0 10px 40px 0 rgba(113, 124, 137, 0.2);
            border-radius: 7px;
            overflow: hidden;
            transition: transform 0.4s ease-out, box-shadow 0.4s ease-out;

            .background {
                border-radius: 7px;
                overflow: hidden;
            }
            .content {
                position: relative;
            }
            /* wwManager:start */

            .contextmenu {
                position: absolute;
                top: 0;
                left: 0;
                //transform: translate(-50%, -50%);
                width: 30px;
                height: 30px;
                color: white;
                background-color: #ef811a;
                border-radius: 100%;
                display: flex;
                justify-content: center;
                align-items: center;
                font-size: 1.2rem;
                cursor: pointer;
                z-index: 2;
            }
            /* wwManager:end */
        }
        .thumbnail-container:first-child {
            margin-left: 25px;
        }
    }
}

.content-dots-wrapper {
    display: flex;
    list-style: none;
    justify-content: center;
    position: relative;
    padding-bottom: 15px;
    margin-top: 20px;

    .content-dot {
        margin-right: 15px;
        border-radius: 100%;
        border-width: 2px;
        border-style: solid;
        .dot {
            cursor: pointer;
            width: 15px;
            height: 15px;
            pointer-events: all;
        }
    }
}
.hidden-mobile {
    display: none;
    @media (min-width: 768px) {
        display: block;
    }
}
.mobile-wrapper {
    display: block;
    position: relative;
    @media (min-width: 768px) {
        display: none;
    }
}
</style>
