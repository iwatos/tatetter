<template>
  <div clas="content">
    <div class="siteInformation">
      <p class="title">縦ったー</p>
      <p>縦書きでツイートできます。</p>
      <p>（Twitterアカウント認証不要）</p>
    </div>

    <div class="sendForm">
      <textarea v-model="text" :placeholder="'こちらに文章を入力すると\n下枠内で縦書きで表示され\nそのままつぶけます。'"></textarea>
      <textarea v-model="verticalText"></textarea>
      <br>
      <a :href="hreftext">つぶやく</a>
    </div>

    <div class="notes">
      <p>
        ＜注意事項＞<br>
        縦書きに変換される際、以下の文字が変更されます。
      <ul>
        <li>半角英数字 → 全角英数字</li>
        <li>半角スペース → 全角スペース</li>
        <li>ー(伸ばし棒) → ｜(パイプ)</li>
      </ul>
        また、字一つひとつの向きは変わりません。<br><br>
        作成者：岩都&nbsp;&nbsp;&nbsp;&nbsp;<a href="https://iwato.netlify.com">HP</a>&nbsp;&nbsp;&nbsp;&nbsp;<a href="https://twitter.com/iwato_s">Twitter(@iwato_s)</a>
      </p>
    </div>
  </div>
</template>
<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      text: ""
    }
  },
  methods: {
    getMatrixSize: function (matrix) {
      const height = matrix.length
      const width = matrix[0].length
      const size = new Array(height, width)
      matrix.forEach(element => {
        if(element != width){
          return new Array(-1, -1)
        }         
      });
      return size
    },

    h2v: function(text) {
      //英数字を全角にする
      text = text.replace(/[A-Za-z0-9]/g, function(s) {return String.fromCharCode(s.charCodeAt(0) + 65248)})
      text = text.replace(/ /g,"　")
      text = text.replace(/ー/g,"｜")

      //文字列の縦横の最大文字数を取得
      const height = text.split("\n").length
      var lengths = new Array(height)
      text.split("\n").forEach(function(element, index){
        lengths[index] = element.length
      });
      const width = Math.max(...lengths)

      //二次元配列で初期化
      var horizonText = new Array(height);
      for(let i = 0; i < height; i++) {
        horizonText[i] = new Array(width).fill("　")
      }

      //一字ずつ二次元配列に格納  
      text.split("\n").forEach(function(e, i){
        e.split("").forEach(function(f, j){
          horizonText[i][j] = e[j]
        })
      })

      var verticalText = new Array(width);
      for(let i = 0; i < width; i++) {
        verticalText[i] = new Array(height).fill("　")
      }

      for (let i = 0; i < height; i++) {
            for (let j = 0; j < width; j++) {                
                verticalText[j][i] = horizonText[height-i-1][j];
            }
        }

      verticalText.forEach(function (e,i,v) {
        v[i] = e.join("")
      })
      verticalText = verticalText.join("\n")

      return verticalText;
    }
  },
  computed: {
    verticalText: function () {
      return this.h2v(this.text)
    },
    hreftext: function () {
      let text = "text=" + this.verticalText.replace(/\n/g, "%0A") +"%0A%0A"
      let url = "url=" + "https://tatetter.netlify.com"
      let hashtags = "hashtags=" + "縦ったー"
      return "https://twitter.com/share?" + text + "&" + url + "&" + hashtags
    }
  }
}
</script>


<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.content {
  display: inline-block;
}
.title {
  font-size: 40px;
}
.siteInformation {
  text-align: center;
  margin: 40px;
}
.sendForm {
  display: block;
  text-align: center;
  textarea {
    font-size: 21px;
    border: medium solid #aaaaaa;
    line-height: 1em;
    text-align: left;
    width: 60%;
    height: 200px;
    margin: 10px;
    resize: vertical;
  };
  a {
    display: inline-block;
    font-size: 30px;
    padding: 0px 10px;
    text-decoration: none;
    border: 1px solid #aaaaaa;
    background-color: #2c3e50;
    color: #ffffff;
  };
}
.notes {
  display: flex;
	justify-content: center;
  margin: 30px 0;
}
</style>
