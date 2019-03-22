<template>
  <div clas="content">
    <div class="siteInformation">
      <p class="title">縦ったー</p>
      <p>縦書きでツイートできます。</p>
      <p>（Twitterアカウント認証不要）</p>
    </div>

    <div class="sendForm">
      <textarea v-model="text" :placeholder="'こちらに文章を入力すると'"></textarea>
      <textarea readonly v-model="verticalText" :placeholder="'こちらに縦書きで表示され、そのままつぶやけます'"></textarea>
      <br>
      <a :href="hreftext" target="_blank">つぶやく</a>
    </div>

    <div class="notes">
      <p>
        ＜注意事項＞<br>
        縦書きに変換される際、以下の文字が変更されます。
      <ul>
        <li>半角文字 → 全角文字</li>
        <li>ー(伸ばし棒) → ｜(パイプ)</li>
      </ul>
        また、字一つひとつの向きは変わりません。<br><br>
        作成者：岩都&nbsp;&nbsp;&nbsp;&nbsp;<a href="https://iwato.netlify.com" target="_blank">HP</a>&nbsp;&nbsp;&nbsp;&nbsp;<a href="https://twitter.com/iwato_s" target="_blank">Twitter(@iwato_s)</a>
      </p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Content',
  data () {
    return {
      text: ""
    }
  },
  methods: {
    
    /**
      * 文字列の高さと最大幅を算出します。
      * @param {String} 幅と高さを知りたい文字列
      * @return {Array} {高さ、幅}の配列 
      */
    getMatrixSize: function (str) {
      let splitedStr = str.split("\n")
      const height = splitedStr.length
      let lengths = new Array(height)
      splitedStr.forEach(function(element, index){
        lengths[index] = element.length
      });
      const width = Math.max(...lengths)
      const size = new Array(height, width)
      return size
    },

    /**
      * 文字列の半角文字を全角文字に変換する
      * @param {String} 半角文字を含んだ文字列
      * @return {String} 全角文字に変換された文字列
      */
     halfToFull: function(halfText) {
       let fullText = halfText
       fullText = fullText.replace(/[!-~]/g, function(s) {return String.fromCharCode(s.charCodeAt(0) + 65248)})
       fullText = fullText.replace(/ /g,"　")
       return fullText
     },

     
    /**
      * 文字列を一文字ずつの二次元配列に変換する
      * @param {String} 文字列
      * @return {Array} 二次元配列
      */
     stringToMatrix: function(str) {
       let size = this.getMatrixSize(str)
       const height = size[0]
       const width = size[1]

       //二次元配列を作成し初期化
       let matrix = new Array(height);
       for(let i = 0; i < height; i++) {
        matrix[i] = new Array(width).fill("　")
       }
       
       //一字ずつ二次元配列に格納  
       str.split("\n").forEach(function(e, i){
         e.split("").forEach(function(f, j){
           matrix[i][j] = e[j]
        })
       })
       
       return matrix
     },

    /**
      * 長方形の二次元配列を時計回り90度回転させる
      * @param {Array} 回転前の二次元配列
      * @return {Array} 回転後の二次元配列
      */
     rotateMatrix: function(matrix) {
      //高さ、幅の取得
      const height = matrix.length
      const width = matrix[0].length

      //幅が0、すなわち空文字の時は終了
      if(width <= 0){
        return matrix
      }

      //二次元配列の初期化
      let rotatedMatrix = new Array(width);
      for(let i = 0; i < width; i++) {
        rotatedMatrix[i] = new Array(height).fill("　")
      }

      //回転させて値を挿入
      for (let i = 0; i < height; i++) {
            for (let j = 0; j < width; j++) {                
                rotatedMatrix[j][i] = matrix[height-i-1][j];
            }
        }
      return rotatedMatrix
     },

    /**
      * 二次元配列を文字列に変換する
      * @param {Array} 二次元配列
      * @return {String} 文字列
      */
     matrixToString: function(matrix) {
      matrix.forEach(function (e,i,v) {
        v[i] = e.join("")
      })
      let str = matrix.join("\n")
      return str
     },

    /**
      * 文字列を縦書きに変換する
      * @param {String} 文字列
      * @return {String} 文字列
      */
    horizonToVertical: function(text) {
      let fullText = this.halfToFull(text).replace(/ー/g,"｜")//半角を全角にし"ー"を"|"に置換する
      let horizonMatrix = this.stringToMatrix(fullText)//文字列を二次元配列に変換
      let verticalMatrix = this.rotateMatrix(horizonMatrix)//二次元配列を時計回り90度回転
      let verticalText = this.matrixToString(verticalMatrix)
      return verticalText;
    },

    /**
      * 文字列先頭の空白を"．"に変換する
      * @param {String} 文字列
      * @return {String} 文字列
      */
    blankToDot: function(str){
      let s = str.slice(0, 1)
      if(s == "　"){
        return "．" + str.slice(1) 
      }
      return str
    }
  },
  computed: {
    /**
      * 縦書き文字列を算出する
      * @return {String} 縦書き文字列
      */
    verticalText: function () {
      let text = this.horizonToVertical(this.text)
      text = this.blankToDot(text)
      return text
    },
    /**
      * ツイートページ遷移用のURLを算出する
      * @return {String} 縦書き文字列
      */
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
    width: 90%;
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
