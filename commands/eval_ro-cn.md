这是 的只读变体`EVAL`命令，该命令无法执行修改数据的命令。

有关何时使用此命令与`EVAL`，请参阅[只读脚本](/docs/manual/programmability/#read-only_scripts).

有关以下内容的更多信息`EVAL`脚本请参考[评估脚本简介](/topics/eval-intro).

@examples

    > SET mykey "Hello"
    OK

    > EVAL_RO "return redis.call('GET', KEYS[1])" 1 mykey
    "Hello"

    > EVAL_RO "return redis.call('DEL', KEYS[1])" 1 mykey
    (error) ERR Error running script (call to b0d697da25b13e49157b2c214a4033546aba2104): @user_script:1: @user_script: 1: Write commands are not allowed from read-only scripts.
