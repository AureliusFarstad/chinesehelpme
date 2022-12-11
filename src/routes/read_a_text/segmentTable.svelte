<script lang="ts">
import pinyin from "pinyin-tone";

type CEDICTEntry = {
    definitions: string[],
    pinyin: string
}

type Segment = {
    ws: string;
    pos: string[];
    def: CEDICTEntry[];
}

export let sortedSegments: Segment[] = []

type tagKeys = {
    [key: string]: string;
}
const tagDict: tagKeys = {
    "A": "non-predicative adjective", // 非謂形容詞
    "Caa": "parallel conjunction", // 對等連接詞
    "Cab": "conjunction, such as: etc.", // 連接詞，如：等等
    "Cba": "conjunction, such as: if", // 連接詞，如：的話
    "Cbb": "relational conjunction", // 關聯連接詞
    "D": "adverb", // 副詞
    "Da": "quantitative adverb", // 數量副詞
    "Dfa": "verb antecedent adverb", // 動詞前程度副詞
    "Dfb": "verb posterior adverb", // 動詞後程度副詞
    "Di": "tense marker", // 時態標記
    "Dk": "sentential adverb", // 句副詞
    "DM": "quantitative form", // 定量式
    "I": "exclamation", // 感嘆
    "Na": "common noun", // 普通名詞
    "Nb": "proper noun", // 專有名詞
    "Nc": `place word`, // 地方詞
    "Ncd": "position word", // 位置詞
    "Nd": "time word", // 時間詞
    "Nep": "referring definite article", // 指代定詞
    "Neqa": "quantitative definite article", // 數量定詞
    "Neqb": "post-quantitative definite article", // 後置數量定詞
    "Nes": "specific definite article", // 特指定詞
    "Neu": "numeral definite article", // 數詞定詞
    "Nf": "measure word", // 量詞
    "Ng": "postposition", // 後置詞
    "Nh": "pronoun", // 代名詞
    "Nv": "nominalized verb", // 名物化動詞
    "P": "preposition", // 介詞
    "T": "particle", // 語助詞
    "VA": "intransitive action verb", // 動作不及物動詞
    "VAC": "causative action verb", // 動作使動動詞
    "VB": "transitive action verb", // 動作類及物動詞
    "VC": "ditransitive action verb", // 動作及物動詞
    "VCL": "locative action verb", // 動作接地方賓語動詞
    "VD": "double object verb", // 雙賓動詞
    "VF": "verb-object verb", // 動作謂賓動詞
    "VE": "verb-sentence verb", // 動作句賓動詞
    "VG": "classification verb", // 分類動詞
    "VH": "intransitive state verb", // 狀態不及物動詞
    "VHC": "causative state verb", // 狀態使動動詞
    "VI": "transitive state verb", // 狀態類及物動詞
    "VJ": "ditransitive state verb", // 狀態及物動詞
    "VK": "sentence-object state verb", // 狀態句賓動詞
    "VL": "verb-object state verb", // 狀態謂賓動詞
    "V_2": "have", // 有
    "DE": "of, by, from", // 的之得地
    "SHI": "is", // 是
    "FW": "foreign language", // 外文
    "COLONCATEGORY": "colon", // 冒號
    "COMMACATEGORY": "comma", // 逗號
    "DASHCATEGORY": "dash", // 破折號
    "DOTCATEGORY": "period", // 點號
    "ETCCATEGORY": "ellipsis", // 刪節號
    "EXCLAMATIONCATEGORY": "exclamation mark", // 驚嘆號
    "PARENTHESISCATEGORY": "parenthesis", // 括號
    "PAUSECATEGORY": "pause", // 頓號
    "PERIODCATEGORY": "full stop", // 句號
    "QUESTIONCATEGORY": "question mark", // 問號
    "SEMICOLONCATEGORY": "semicolon", // 分號
    "SPCHANGECATEGORY": "double vertical line", // 雙直線
    "WHITESPACE": "whitespace" // 空白
}

const tagColors: tagKeys = {
    "A": "#FFB6C1", // 非謂形容詞
    "Caa": "#D8BFD8", // 對等連接詞
    "Cab": "#D8BFD8", // 連接詞，如：等等
    "Cba": "#D8BFD8", // 連接詞，如：的話
    "Cbb": "#D8BFD8", // 關聯連接詞
    "D": "#E6E6FA", // 副詞
    "Da": "#E6E6FA", // 數量副詞
    "Dfa": "#E6E6FA", // 動詞前程度副詞
    "Dfb": "#E6E6FA", // 動詞後程度副詞
    "Di": "#E6E6FA", // 時態標記
    "Dk": "#E6E6FA", // 句副詞
    "DM": "#E6E6FA", // 定量式
    "I": "#FFFFE0", // 感嘆
    "Na": "#90EE90", // 普通名詞
    "Nb": "#90EE90", // 專有名詞
    "Nc": "#DEBACE", // 地方詞
    "Ncd": "#DEBACE", // 位置詞
    "Nd": "#F08080", // 時間詞
    "Nep": "#FFE1E1", // 指代定詞
    "Neqa": "#FFE1E1", // 數量定詞
    "Neqb": "#FFE1E1", // 後置數量定詞
    "Nes": "#FFE1E1", // 特指定詞
    "Neu": "#FFE1E1", // 數詞定詞
    "Nf": "#C8FFD4", // 量詞
    "Ng": "#F29393", // 後置詞
    "Nh": "#FFCCB3", // 代名詞
    "Nv": "#90A17D", // 名物化動詞
    "P": "#BA94D1", // 介詞
    "T": "#BA94D1", // 語助詞
    "VA": "#ADD8E6", // 動作不及物動詞
    "VAC": "#ADD8E6", // 動作使動動詞
    "VB": "#ADD8E6", // 動作類及物動詞
    "VC": "#ADD8E6", // 動作及物動詞
    "VCL": "#ADD8E6", // 動作接地方賓語動詞
    "VD": "#ADD8E6", // 雙賓動詞
    "VF": "#ADD8E6", // 動作謂賓動詞
    "VE": "#ADD8E6", // 動作句賓動詞
    "VG": "#ADD8E6", // 分類動詞
    "VH": "#ADD8E6", // 狀態不及物動詞
    "VHC": "#ADD8E6", // 狀態使動動詞
    "VI": "#ADD8E6", // 狀態類及物動詞
    "VJ": "#ADD8E6", // 狀態及物動詞
    "VK": "#ADD8E6", // 狀態句賓動詞
    "VL": "#ADD8E6", // 狀態謂賓動詞
    "V_2": "#ADD8E6", // 有
    "DE": "#98FB98", // 的之得地
    "SHI": "#FFDAB9", // 是
    "FW": "lightgray", // 外文
    "COLONCATEGORY": "lightgray", // 冒號
    "COMMACATEGORY": "lightgray", // 逗號
    "DASHCATEGORY": "lightgray", // 破折號
    "DOTCATEGORY": "lightgray", // 點號
    "ETCCATEGORY": "lightgray", // 刪節號
    "EXCLAMATIONCATEGORY": "lightgray", // 驚嘆號
    "PARENTHESISCATEGORY": "lightgray", // 括號
    "PAUSECATEGORY": "lightgray", // 頓號
    "PERIODCATEGORY": "lightgray", // 句號
    "QUESTIONCATEGORY": "lightgray", // 問號
    "SEMICOLONCATEGORY": "lightgray", // 分號
    "SPCHANGECATEGORY": "lightgray", // 雙直線
    "WHITESPACE": "lightgray" // 空白
}
</script>
<div class="scroll">
<div class="toggled-segments-table">
    <div class="header index-header"></div>
    <div class="header word-header">Word</div>
    <div class="header dictionary-header">Dictionary</div>
    <div class="header part-of-sentence-header">Part of Sentence</div>

    {#each sortedSegments as s, i}
        <div class="count row">{i}</div>
        <div class="word row">{s.ws}</div>
        <div class="dictionary row">
            {#if s.def}
                {#each s.def as dict_entry, i}
                    <div class="dictionary-entry">
                        <span class="pinyin">{pinyin(dict_entry.pinyin.toLowerCase())}</span>
                        <span class="definition">{dict_entry.definitions.join(', ')}</span>
                    </div>
                {/each}
            {/if}
        </div>
        <div class="part-of-sentence row">
            {#each s.pos as tag}
                <span 
                    class="tag" 
                    style:background-color="{tagColors[tag]}">
                    {tagDict[tag]}
                </span>
            {/each}
        </div>
    {/each}
</div>
</div>
{#if sortedSegments.length == 0}
<div class="help">Select words from the text.</div>
{/if}

<style>
.scroll {
    overflow: scroll;
}
.toggled-segments-table {
  display: grid;
  grid-template-columns: auto auto 10fr 6fr;
  grid-column-gap: 8px;
  grid-template-rows: 28px repeat(auto-fill, minmax(8px, auto));

  margin-top: 10px;
  margin-left: 10px;
  margin-right: 10px;

  overflow: scroll;  /* overflow condition on parent */
}

.header  {
  border-bottom: 1px solid #FF9999;
  padding-right: 0px;
  padding-left: 0px;  
  
  font-family: 'Futura';
  font-size: 18px;
  font-weight: 800px;
}

.help {
    color: gray;
    font-family: 'Futura';
    font-weight: 800px;
    font-size: 18px;
    margin-left: 10px;
}

.index-header {
    padding-right: 6px;
    padding-left: 6px;
    min-width: 14px;
}
.word-header {
    padding-right: 6px;
    padding-left: 6px;
    min-width: 70px;
}

.row {
  border-bottom: 1px solid #FF9999;
  padding-right: 4px;
  padding-left: 4px; 
}

.count {
  font-family: serif;
  font-size: 12px;
  line-height: 27px;
}

.word {
  padding-right: 6px;
  padding-left: 6px;
  padding-bottom: 2px;
  min-width: 70px;
  
  font-family: 'STSong', sans-serif;
  font-size: 20px;
  line-height: 27px;
}

.dictionary-entry {
    padding-bottom: 5px;
}
.dictionary {
  font-size: 18px;
  padding-top: 3px;
  /* padding-bottom: 6px; */
}

.pinyin {
  font-size: 16px;
  font-family: 'Georgia', 'Palatino';
  font-style: bold;
  font-weight: 800;
}
.definition {
  font-size: 16px;
  padding-left: 10px;
}

.part-of-sentence {
}
.tag {
    white-space: nowrap;
    /* display: inline-block; */
  border-radius: 8px;
  margin-top: 2px;
  margin-right: 4px;
  padding-left: 6px;
  padding-right: 6px;
  padding-top: 3px;
  padding-bottom: 3px;
  height: 26px;
  width: 26px;

  font-size: 12px;
  line-height: 26px;
  font-family: 'helvetica', 'arial';
  font-weight: 200;
  text-align: center;
}

</style>