<template>
    <div id="app">
        <div class="slide-bar">
            <user></user>
            <list></list>
        </div>
        <div class="main">
            <notice></notice>
            <message></message>
            <user-text></user-text>
        </div>

    </div>
</template>

<script>
    import User from './components/User.vue';
    import List from './components/List.vue';
    import Message from './components/Message.vue';
    import UserText from './components/Text.vue';
    import Notice from './components/Notice.vue';

    export default {
        name: 'app',
        data () {
            return {
                messageDiv: ''
            }
        },
        components: {
            User, UserText, Message, List, Notice
        },
        created: function () {
            let _this = this;
            let conn = new WebSocket('ws://127.0.0.1:9501');

            conn.onopen = function (evt) {
                let data = {
                    'msg': ' 连接成功！',
                    'type': 'success'
                }
                _this.$store.dispatch('showNotice', data)
                _this.$store.dispatch('changeStatus', true)
            }

            conn.onclose = function (evt) {
                let data = {
                    'msg': ' 已断开连接！',
                    'type': 'error'
                }
                _this.$store.dispatch('showNotice', data)
                _this.$store.dispatch('changeStatus', false)
            }

            conn.onmessage = function (evt) {
                let msg = JSON.parse(evt.data);
                switch (msg.type) {
                    case 'connect':
                        _this.$store.dispatch('addUser', msg.data)
                        var data = {
                            'msg': msg.data.nickname + '已进入',
                            'type': 'success'
                        }
                        _this.$store.dispatch('showNotice', data)
                        _this.$store.dispatch('setCount', msg.data.count)
                        break;
                    case 'disconnect':
                        _this.$store.dispatch('removeUser', msg.data.id)
                        var data = {
                            'msg': msg.data.nickname + '已离开',
                            'type': 'warning'
                        }
                        _this.$store.dispatch('showNotice', data)
                        _this.$store.dispatch('setCount', msg.data.count)
                        break;
                    case 'self_init':
                        _this.$store.dispatch('setUser', msg.data)
                        _this.$store.dispatch('setCount', msg.data.count)
                        break;
                    case 'other_init':
                        _this.$store.dispatch('addUser', msg.data)
                        break;
                    case 'message':
                        _this.$store.dispatch('addMessage', msg.data).then(() => {
                                    this.messageDiv = document.querySelector('.message');
                                    this.messageDiv.scrollTop = this.messageDiv.scrollHeight + 10;
                                }
                        )
                        break;
                }
            }
            _this.$store.dispatch('setConn', conn)
        },
    }
</script>

<style lang="less">
    #app {
        width: 800px;
        height: 600px;
        position: absolute;
        top: 50%;
        left: 50%;
        margin-left: -400px;
        margin-top: -300px;

        display: flex;
        flex-wrap: nowrap;
        flex-direction: row;
        border-radius: 3px;

    .slide-bar, .main {
        height: 100%;
    }

    .slide-bar {
        width: 200px;
        background: #2e3238;
        color: #f4f4f4;
    }

    .main {
        width: 600px;
        background: #eee;
        display: flex;
        flex-direction: column;
    }

    }
</style>