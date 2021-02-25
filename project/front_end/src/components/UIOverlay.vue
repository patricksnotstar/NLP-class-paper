<template>
    <div class="UIOverlay">
        <div class="bottom-left">
            <div
                class="category"
                v-for="cat in categories"
                :key="cat.category"
                @click="filter(cat)"
                :style="[
                    cat.include
                        ? { backgroundColor: cat.bgColor, color: 'black' }
                        : { backgroundColor: null },
                ]"
            >
                <div
                    class="circle"
                    :style="{ backgroundColor: cat.color }"
                ></div>
                <div class="category-title">{{ cat.category }}</div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: "UIOverlay",
    data() {
        return {
            categories: [
                {
                    category: "Politics",
                    color: "#fa8d4f",
                    index: 0,
                    include: false,
                    realCategory: "politics",
                    bgColor: "rgb(251,141,79)",
                },
                {
                    category: "Business",
                    color: "#fdd742",
                    index: 1,
                    include: false,
                    realCategory: "business",
                    bgColor: "rgb(253,215,66)",
                },
                {
                    category: "Sports",
                    color: "#a3e048",
                    index: 2,
                    include: false,
                    realCategory: "sports",
                    bgColor: "rgb(163,224,72)",
                },
                {
                    category: "Entertainment",
                    color: "#2EC582",
                    index: 3,
                    include: false,
                    realCategory: "entertainment",
                    bgColor: "rgb(46,197,130)",
                },
                {
                    category: "Technology",
                    color: "#50d4fe",
                    index: 4,
                    include: false,
                    realCategory: "technology",
                    bgColor: "rgb(80,212,254)",
                },
                {
                    category: "Good News",
                    color: "#A71EF7",
                    index: 5,
                    include: false,
                    realCategory: "Positive",
                    bgColor: "#A71EF7",
                },
                {
                    category: "Bad News",
                    color: "#CA3B3B",
                    index: 6,
                    include: false,
                    realCategory: "Negative",
                    bgColor: "#CA3B3B",
                },
                {
                    category: "Neutral News",
                    color: "#909681",
                    index: 7,
                    include: false,
                    realCategory: "Neutral",
                    bgColor: "#909681",
                }
            ],
            active_filters: [0, 0, 0, 0, 0, 0, 0, 0, 0],
        };
    },
    methods: {
        filter(cat) {
            if (cat.category == "Good News"){
                this.categories[5].include = !this.categories[5].include;
                this.categories[6].include = false;
                this.categories[7].include = false;
            }
            else if (cat.category == "Bad News"){
                this.categories[5].include = false;
                this.categories[6].include = !this.categories[6].include;
                this.categories[7].include = false;
            }
            else if (cat.category == "Neutral News"){
                this.categories[5].include = false;
                this.categories[6].include = false;
                this.categories[7].include = !this.categories[7].include;
            }
            for (let i in this.categories) {
                if (cat.category === this.categories[i].category && cat.category != "Good News" && cat.category != "Bad News" && cat.category != "Neutral News") {
                    this.categories[i].include = !this.categories[i].include;
                }
            }
            // for (let i in this.categories) {
            //     if (cat.category === this.categories[i].category ) {
            //         this.categories[i].include = !this.categories[i].include;
            //     }
            // }
            this.$emit("revaluate-filters", this.categories);
        },
    },
    mounted() {
        this.$emit("revaluate-filters", this.categories);
    },
};
</script>



<style scoped>
.UIOverlay {
    position: absolute;
    pointer-events: none;
    width: 100%;
    height: 100%;
}
.top-right {
    pointer-events: auto;
    position: absolute;
    top: 10px;
    right: 50px;
}
.bottom-left {
    pointer-events: auto;
    position: absolute;
    bottom: 50px;
    left: 10px;
}
.top-centre {
    pointer-events: auto;
    position: absolute;
    left: 40%;
    top: 10px;
}
.button {
    border: 0.1px solid #8da9bf;
    letter-spacing: 0.4px;
    background-color: rgb(255, 255, 255);
    color: #ffffff;
    box-shadow: 0 3px 6px #3535353b;
    border-radius: 5px;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 6px 12px;
    user-select: none;
    cursor: pointer;
    margin-bottom: 5px;
    transition: all;
    transition-duration: 150ms;
}
.button:hover {
    background-color: #f2f2f2;
    background-color: #cbd5dc;
    color: #414141;
}
.category {
    border: 0.1px solid #8da9bf;
    letter-spacing: 0.4px;
    background-color: rgb(255, 255, 255, 0.05);
    color: #fcfafa;
    box-shadow: 0 3px 6px #3535353b;
    border-radius: 5px;
    display: flex;
    justify-content: flex-start;
    align-items: center;
    padding: 6px 12px;
    user-select: none;
    cursor: pointer;
    transition: all;
    transition-duration: 150ms;
    margin-bottom: 5px;
    width: 100%;
}
.category:hover {
    background-color: #cbd5dc;
    color: #414141;
}
.category-title {
    opacity: 1;
}
.category-title {
    opacity: 1;
}
.circle {
    border-radius: 50%;
    width: 15px;
    height: 15px;
    margin-right: 10px;
}
.close {
    color: #aaa;
    float: right;
    font-size: 28px;
    font-weight: bold;
}
.close:hover,
.close:focus {
    color: black;
    text-decoration: none;
    cursor: pointer;
}
input[type="text"] {
    border: none;
    box-shadow: none;
    outline: none;
    letter-spacing: 0.4px;
    background-color: transparent;
    border-radius: 5px;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 6px 12px;
    user-select: none;
    cursor: pointer;
    transition: all;
    transition-duration: 150ms;
    font-family: -apple-system, "Avenir", Helvetica, Arial, sans-serif;
    color: #414141;
    font-size: 16px;
}
input[type="text"]:hover {
    cursor: pointer;
    background-color: #f2f2f2;
}
input[type="text"]:focus {
    outline: none;
    border: 1px solid #8da9bf;
}
::placeholder {
    display: none;
    font-family: -apple-system, "Avenir", Helvetica, Arial, sans-serif;
    color: #414141;
    font-size: 16px;
    text-align: left;
}

img {
    width: 180px;
    height: auto;
}
@import url("//maxcdn.bootstrapcdn.com/font-awesome/4.1.0/css/font-awesome.min.css");
.search {
    position: relative;
}
.search input {
    text-indent: 0px;
}
.search .fa-search {
    position: absolute;
    top: 8px;
    right: 7px;
    font-size: 15px;
}
h2 {
    text-align: center;
    margin-bottom: 30px;
}
a {
    text-decoration: none;
}
a:hover {
    text-decoration: underline;
    color: #cbd5dc;
}
</style>
