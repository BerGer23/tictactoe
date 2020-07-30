<template>
<div>
    <div class="tableContainer">
        <table class="field" >
            <tbody>
            <tr v-for="(row,rowindex) in rows" v-bind:key="rowindex">
                <td v-for="(col,colindex) in row" v-bind:key="rowindex-colindex" @click="selectValue(rowindex,colindex,true)">
                    <Field :value="col"/>
                </td>
            </tr>
        </tbody>
        </table>
    </div>
    <div class="gameState">
        {{gameStateLabel}}
    </div>
    <button v-if="gameState!=0" @click="reset()">Neu starten</button>
</div>
</template>

<script>
import Field from './Field.vue'

export default {
  name: 'PlayingField',
  components: {
      Field
  },
  data: function () {
    return this.initialState();
  },
  computed: {
      gameStateLabel: function() {
          if(this.gameState == 0)
            return "";
          else if(this.gameState==1){
            return "Ein überragender Sieg für ANGRY"
          }
          else if(this.gameState==2){
            return "Mit viel Glück hat HAPPY gewonnen"
          } else {
            return "Eigentlich hätte ANGRY es deutlich mehr verdient, aber leider nur Unentschieden."
          }
      }
  },
  methods: {
    initialState: function () {
        return {
            rows : [
            [
                0, 0, 0
            ],
            [
                0, 0, 0
            ],
            [
                0, 0, 0
            ]
            ],
            gameState: 0,
            xTurn: true
        }
    },
    reset: function () {
        Object.assign(this.$data, this.initialState())
    },
    evaluateField: function(rowArray) {
        var rowSum = [0,0,0]
        var colSum = [0,0,0]
        var diaSum = [0,0]
        var everyFieldTaken = true
        
        for(let i=0; i<rowArray.length; i++){
            for(let j=0; j<rowArray[i].length; j++){
                if(i==j) {
                    diaSum[0] += rowArray[i][j]
                }
                if(i+j==2){
                    diaSum[1] +=  rowArray[i][j]
                }
                rowSum[i]+=rowArray[i][j]
                colSum[j]+=rowArray[i][j]
                if(rowArray[i][j]==0) everyFieldTaken = false;
            }
        }

        return {
            rowSum: rowSum,
            colSum: colSum,
            diaSum: diaSum,
            everyFieldTaken: everyFieldTaken
        }
    },
    determineGameState: function(state){
        for (const row of state.rowSum){
            if(row==3)
            {
                this.gameState = 1;
                return;
            } else if (row==30){
                this.gameState = 2;
                return;
            }
        }
        for (const col of state.colSum){
            if(col==3)
            {
                this.gameState = 1;
                return;
            } else if (col==30){
                this.gameState = 2;
                return;
            }
        }
        for(const dia of state.diaSum){
            if(dia==3)
            {
                this.gameState = 1;
                return;
            } else if (dia==30){
                this.gameState = 2;
                return;
            }
        }
        if(state.everyFieldTaken==true){
            this.gameState = 3;
            return;
        }
    },
    getRandomInt: function (max) {
        return Math.floor(Math.random() * Math.floor(max));
    },
    randomEmptyField: function() {
        for(let i=0; i<this.rows.length; i++){
            for(let j=0; j<this.rows[i].length; j++){
                if(this.rows[i][j]==0){
                    return {
                        rowIndex: i,
                        colIndex: j
                    }
                }
            }
        }
    },
    stateDelta: function(newState, oldState){
        var rowDelta = [0,0,0]
        var colDelta = [0,0,0]
        var diaDelta = [0,0]
        var rowSum = 0
        var colSum = 0
        var diaSum = 0
        
        
        for(let i = 0; i< newState.rowSum.length; i++){
            rowDelta[i] = newState.rowSum[i] - oldState.rowSum[i]
            if(newState.rowSum[i]==3){ 
                rowSum += 1000
            }else if(newState.rowSum[i]==21 && oldState.rowSum[i]==20){
                rowSum += 100
            }else if(newState.rowSum[i]==11 && oldState.rowSum[i]==10){
                rowSum += 1
            }else {
                rowSum += rowDelta[i]
            }
            colDelta[i] = newState.colSum[i] - oldState.colSum[i]
             if(newState.colSum[i]==3){ 
                colSum += 1000
            }else if(newState.colSum[i]==21 && oldState.colSum[i]==20){
                colSum += 100
            }else if(newState.colSum[i]==11 && oldState.colSum[i]==10){
                colSum += 1    
            }else {
                colSum += colDelta[i]
            }
            colSum += colDelta[i]
            if( i < newState.diaSum.length){
                diaDelta[i] = newState.diaSum[i] - oldState.diaSum[i]
                
                if(newState.diaSum[i]==3){ 
                    diaSum += 1000
                }else if(newState.diaSum[i]==21 && oldState.diaSum[i]==20){
                    diaSum += 100
                }else if(newState.colSum[i]==11 && oldState.colSum[i]==10){
                    colSum += 1
                }else {
                    diaSum += diaDelta[i]
                }
            }
        }
        
        return {
            rowDelta: rowDelta,
            rowSum: rowSum,
            colDelta: colDelta,
            colSum: colSum,
            diaDelta: diaDelta,
            diaSum: diaSum,
            totalSum: rowSum+colSum+diaSum
        }
    },
    findOptimalField: function() {
        var rowArray = this.rows.map(function(arr) {
            return arr.slice();
        });
        var originalState= this.evaluateField(rowArray)
        var bestMoves=[];
        for(let i=0; i<rowArray.length; i++){
            for(let j=0; j<rowArray[i].length; j++){
                if(rowArray[i][j]==0){
                    var rowArrayCopy = this.rows.map(function(arr) {
                        return arr.slice();
                    });
                    rowArrayCopy[i][j] = 1
                    var newState = this.evaluateField(rowArrayCopy)
                    var delta = this.stateDelta(newState, originalState)

                    if(bestMoves.length== 0 || delta.totalSum >= bestMoves[0].totalSum){
                        if(bestMoves.length>0 && bestMoves[0].totalSum<delta.totalSum){
                            bestMoves = []
                        }
                        bestMoves.push(
                            {
                                rowIndex: i,
                                colIndex: j,
                                totalSum: delta.totalSum
                            }
                        )
                    }
                }
            }
        }

        var selectedMove = bestMoves[this.getRandomInt(bestMoves.length)]
        console.log("selected move: "+selectedMove.totalSum + " improvement, row "+selectedMove.rowIndex+ " col "+selectedMove.colIndex )
        return selectedMove
    },
    aiTurn: function() {
        //var turn = this.randomEmptyField()
        var turn = this.findOptimalField()
        var thinkingTime = 300+this.getRandomInt(1500)
        setTimeout(() => this.selectValue(turn.rowIndex, turn.colIndex, false), thinkingTime);
    },
    refreshGameState: function () {
        //0 running, 1 A won, 2 H won, 3 draw
        var state = this.evaluateField(this.rows)
        this.determineGameState(state)
        if(this.xTurn===false&&this.gameState==0) this.aiTurn()
    },
    selectValue: function (rowindex,colindex,manual) {
        if(manual == this.xTurn && this.gameState == 0){
            if(this.rows[rowindex][colindex]==0){ 
                this.xTurn = this.xTurn ? false : true
                const newRow = this.rows[rowindex].slice(0)
                newRow[colindex] = this.xTurn ? 1:10;
                this.$set(this.rows, rowindex, newRow)
            }
            this.refreshGameState()
        }
    },
    
  }
}
</script>
<style>
    .field {
        font-size: 60px;
        margin: auto;
    }
    .table-container{
        /* Text zentrieren */
        text-align:center;
        
        /* Inhalt vertikal mittig platzieren */
        display: -webkit-flex;
        display: -webkit-box;
        display: -ms-flexbox;
        display: flex;
        -webkit-align-items: center;
        -webkit-box-align: center;
        -ms-flex-align: center;
        align-items: center;
    }
</style>