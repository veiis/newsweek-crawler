<template>
  <div id="app">
    <div class="container">
      <div class="top-section">
        <input
          type="text"
          placeholder="What kind of news you are looking for?"
          v-model="qu"
          v-on:keypress.enter="getNewsBySearch(50)"
        />
        <div class="options">
          <select name="count" id>
            <option value="10">10</option>
            <option value="20">20</option>
          </select>
        </div>
      </div>
      <div class="news">
        <div class="news-item" v-for="item in news" :key="item.id">
          <div class="left">
            <img :src="item.img"/>
          </div>
          <div class="right">
            <h3>
              <a :href="item.link">{{ item.title }}</a>
            </h3>
            <p>{{ item.summary }}</p>
            <span>{{ item.date }}</span>
            <span>{{ item.source }}</span>
          </div>
        </div>
      </div>
      :|
    </div>
  </div>
</template>

<script>
const cheerio = require("cheerio");
const fetch = require("node-fetch");
export default {
  name: "app",
  data() {
    return {
      news: [],
      qu: ""
    };
  },
  methods: {
    getNews: function(index) {
      return fetch(
        "https://cors-anywhere.herokuapp.com/https://www.theweek.co.uk/news-feed/page/" +
          index +
          "/0"
      ).then(res => res.text());
    },
    getBySearch: function(index, q) {
      return fetch(
        "https://cors-anywhere.herokuapp.com/https://www.theweek.co.uk/search/site/" +
          q +
          "/page/" +
          index +
          "/0"
      ).then(res => res.text());
    },

    getNewsBySearch: function(counter) {
      this.news = [];
      for (let i = 0; i < counter; i++) {
        this.getBySearch(i, this.qu).then(body => {
          const $ = cheerio.load(body);
          let single = {
            id: i,
            title: "",
            date: "",
            link: "",
            img: "",
            source: "",
            summary: ""
          };
          $("ol.search-results li").each((i, item) => {
            if (
              $(item)
                .find(".search-snippet-info h2 a")
                .text() !== ""
            ) {
              single = {
                title: $(item)
                  .find(".search-snippet-info h2 a")
                  .text(),
                date: $(item)
                  .find(".date")
                  .text(),
                link:
                  "https://www.theweek.co.uk/" +
                  $(item)
                    .find(".content div h2 a")
                    .attr("href"),
                img: $(item)
                  .find(".primary-image a img")
                  .attr("src"),
                source: "The News Week",
                summary: $(item)
                  .find("p .short-teaser")
                  .text()
              };
              this.news.push(single);
            }
            single = {};
          });
        });
      }
    }
  },
  created() {
    for (let i = 0; i < 5; i++) {
      this.getNews(i).then(body => {
        const $ = cheerio.load(body);
        let single = {
          id:i ,
          title: "",
          date: "",
          link: "",
          img: "",
          source: "",
          summary: ""
        };
        $("ul.view-rows li").each((i, item) => {
          if (
            $(item)
              .find(".content div h2 a")
              .text() !== ""
          ) {
            single = {
              title: $(item)
                .find(".content div h2 a")
                .text(),
              date: $(item)
                .find(".content div span")
                .text(),
              link:
                "https://www.theweek.co.uk/" +
                $(item)
                  .find(".content div h2 a")
                  .attr("href"),
              img: $(item)
                .find(".content div a img")
                .attr("src"),
              source: $(item)
                .find(".content .field-name-kicker a")
                .text(),
              summary: $(item)
                .find(".content p")
                .text()
            };
            this.news.push(single);
          }

          single = {};
        });
      });
    }
  }
};
</script>

<style>
@import "./assets/style.css";
</style>
