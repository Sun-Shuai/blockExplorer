<template>

  <div>
    <div style="width: 80%;margin: 0 auto" class="main-content">

      <div style="display: flex;justify-content: space-between;align-items: center">
        <h1 style="margin-top: 20px">Block Explorer</h1>
        <!--12430-blockhash:00000000da61d767c9b8f16855119ed50c62bca72adefa2442ebe1810a4f0cd8-->
        <!--txid:82cb4fb99122133604e91a2ff2120f1baf0825f3f5d4a9d691a5efbcdd1d9372-->
        <div>
          <el-input style="width: 400px" clearable size="mini" v-model="inputHash"
                    placeholder="Search by hash or height">
          </el-input>
          <el-button type="primary" icon="el-icon-search" size="mini" @click="goToDetail(inputHash)"
                     :disabled="inputHash===''">Go!
          </el-button>
        </div>
      </div>

      <el-divider></el-divider>
      <div style="display: flex;justify-content: space-around">

        <div style="width: 320px;display: inline-block">

          <el-card>
            <div>

              <div style="text-align: center">
                <div>Blocks</div>
                <div style="font-size: 3rem;margin-top: 8px">{{blockCount}}</div>
              </div>

              <el-divider></el-divider>

              <div style="text-align: center">
                <div>Time from last block</div>
                <div style="display: flex;justify-content: space-around;margin-top: 8px">
                  <div style="display: inline-block">
                    <div style="font-size: 2.4rem">{{parseInt(lastTime/3600)}}</div>
                    <div>hours</div>
                  </div>
                  <div style="display: inline-block">
                    <div style="font-size: 2.4rem">{{parseInt(lastTime/60)}}</div>
                    <div>minutes</div>
                  </div>
                  <div style="display: inline-block">
                    <div style="font-size: 2.4rem">{{parseInt(lastTime%60)}}</div>
                    <div>seconds</div>
                  </div>
                </div>
              </div>

              <el-divider></el-divider>

              <div class="base-info">
                <span>difficulty</span>
                <span>{{currentDifficulty}}</span>
              </div>

            </div>
          </el-card>


        </div>

        <div style="width:800px;display: inline-block;padding: 12px 36px">
          <h3>Recent Blocks</h3>


          <div>
            <el-timeline>
              <el-timeline-item placement="top" v-for="(block,index) in blockList"
                                :timestamp="timestamp2date(block.time)"
                                :type="index === 0?'success':'info'">
                <el-card shadow="hover">

                  <div style="display: flex;justify-content: space-between;align-items: center">
                    <div style="font-size: 2rem;">{{block.height}}</div>
                    <el-button type="text"
                               size="mini"
                               @click="goToDetail(block.hash)"
                               icon="el-icon-position">
                      Detail
                    </el-button>
                  </div>


                  <div class="base-info">
                    <span>Hash</span>
                    <span>{{block.hash}}</span>
                  </div>

                  <div style="display: flex;justify-content: space-between;margin: 18px 0">
                    <div style="width: 45%;display: inline-block">
                      <div class="base-info">
                        <span>Confirmations</span>
                        <span>{{block.confirmations}}</span>
                      </div>
                      <div class="base-info">
                        <span>Nonce</span>
                        <span>{{block.nonce}}</span>
                      </div>
                      <div class="base-info">
                        <span>Difficulty</span>
                        <span>{{block.difficulty}}</span>
                      </div>
                    </div>
                    <div style="width: 45%;display: inline-block">
                      <div class="base-info">
                        <span>Size</span>
                        <span>{{block.size}} bytes</span>
                      </div>
                      <div class="base-info">
                        <span>Bits</span>
                        <span>0x{{block.bits}}</span>
                      </div>
                    </div>
                  </div>

                  <div class="base-info">
                    <span>Merkleroot</span>
                    <span>{{block.merkleroot}}</span>
                  </div>
                  <div class="base-info">
                    <span>Previousblockhash</span>
                    <span>{{block.previousblockhash}}</span>
                  </div>
                  <div class="base-info">
                    <span>Nextblockhash</span>
                    <span>{{block.nextblockhash}}</span>
                  </div>
                  <div class="base-info">
                    <span>Tx</span>
                    <span>{{block.tx}}</span>
                  </div>

                </el-card>
              </el-timeline-item>

            </el-timeline>
          </div>
        </div>
      </div>

      <el-divider></el-divider>


    </div>

    <myfooter></myfooter>

  </div>

</template>

<script>
  import axios from "axios";
  import {username, password} from '~/assets/rpcconfig'
  import myfooter from '~/components/myfooter.vue'

  export default {
    data() {
      return {
        //part1
        blockCount: '-',
        lastTime: 0,
        currentDifficulty: '-',
        //part2
        blockList: [],
        //search bar
        inputHash: '',
      }
    },
    methods: {
      requestRpc(method, params, callback) {
        axios.post('api', {
          method, params
        }, {
          auth: {
            username: username,
            password: password
          }
        }).then((response) => {
          console.log(method + ': ' + response.data.result);
          callback(response.data.result);
        }).catch((error) => {
          console.log('err:' + error);
        })
      },

      getBlockchainInfo() {
        this.requestRpc('getblockchaininfo', null, (result) => {
          this.blockCount = result.blocks;
          this.currentDifficulty = result.difficulty;

          this.getRecentBlocks();
        });
      },

      getRecentBlocks() {
        let times = this.blockCount;

        for (let i = 0; i < 10; i++) {
          if (times - i < 0) break;
          this.requestRpc('getblockhash', [times - i], (result) => {
            this.getBlockDetail(result);
          })
        }

      },


      getBlockDetail(hash) {
        this.requestRpc('getblock', [hash], (result) => {
          this.blockList.push(result);
          this.blockList.sort(this.compare('height'));
        });
      },

      compare(property) {
        return function (a, b) {
          return b[property] - a[property];
        }
      },

      setLastTime() {
        setInterval(() => {
          this.lastTime = new Date().getTime() / 1000 - this.blockList[0].time;
        }, 1000);
      },

      timestamp2date(timestamp) {
        let date = new Date(timestamp * 1000);
        return date.getFullYear() + '-'
          + (date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) : date.getMonth() + 1) + '-'
          + (date.getDate() < 10 ? '0' + date.getDate() : date.getDate()) + ' '
          + (date.getHours() < 10 ? '0' + date.getHours() : date.getHours()) + ':'
          + (date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes()) + ':'
          + (date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds());
      },

      goToDetail(heightOrHash) {
        if (/^\d+$/.test(heightOrHash) || heightOrHash.startsWith('-')) {//height
          let height = parseInt(heightOrHash);
          if (height < 0 || height > this.blockCount) {
            this.$message({
              duration: 0,
              showClose: true,
              message: 'Height out of range!',
              type: 'error'
            });
            return;
          }
          this.requestRpc('getblockhash', [height], (result) => {
            this.$router.push({name: 'blockdetail', params: {hash: result}});
          })
        } else {//hash
          this.$router.push({name: 'blockdetail', params: {hash: heightOrHash}});
        }
      }
    },
    mounted() {
      this.getBlockchainInfo();
      this.setLastTime();
    }
  }
</script>

<style>
  h3 {
    margin: 12px 0;
  }

  .base-info {
    display: flex;
    justify-content: space-between;
    color: dimgray;
    margin: 5px 0;
  }

  .base-info span:first-child {
    font-weight: bolder;
  }

  .base-info span:nth-child(2) {
    margin-left: 24px;
    overflow: hidden;
    text-overflow: ellipsis;
  }


</style>
