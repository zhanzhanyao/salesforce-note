# 1 create sandbox
    # 连接
    sfdx force:auth:web:login -a DevHub -d
    # 创建
    sfdx force:org:create -t sandbox -f config/dev-sandbox-def.json -u DevHub -a YaoSandbox -w 240 sandboxName=Yao

# 2 check list
    sfdx force:org:list

# 3 open
    sfdx force:org:open -u yao.zhan@rea.spot.yao    