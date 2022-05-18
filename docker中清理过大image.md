```bash
# cd /var/lib/docker/overlay2
# 显示大小超过G的
# du -sh *|grep G
1.1G    d26846eba748521896fb80c5ef13150ebd1785878634c53d0ee7ed56d649ca3e


# for i in `docker ps -q`;do n=`docker inspect $i --format {{.GraphDriver.Data.UpperDir}}`;echo "$i --> $n";done
19821db5cacc --> /var/lib/docker/overlay2/3e8e9191d891e61e5fb62d0fb9174e63df7412674b6ac8f1acb2b8b35ac8910c/diff
97069ce8f167 --> /var/lib/docker/overlay2/d26846eba748521896fb80c5ef13150ebd1785878634c53d0ee7ed56d649ca3e/diff
d8d26423d87d --> /var/lib/docker/overlay2/373a217e437cffff0944f454a5009c33e01cbf0db9860c7a4a30bd8a36b74aa2/diff
a873468f3285 --> /var/lib/docker/overlay2/22f45dfad3d1a47e8c843fbc90a5f82e74fdaca8e4756b28fe3d2137453e781e/diff
04e49104bbe1 --> /var/lib/docker/overlay2/f0caa1c8b096cdcd0ba0747ab3b5982b22349cbd0400d1690bba29f8c3d41623/diff
dc03620e958e --> /var/lib/docker/overlay2/a4a58866fef6602794b5ed4b7d646159e1e52cb6b097a228eba7ce92e64d6a22/diff
58bb98d11cb3 --> /var/lib/docker/overlay2/7fb62d17efecd8749ad857e75f3f1a22da422c3278ba437d49c391bf16d2a405/diff
83f39e2b88d0 --> /var/lib/docker/overlay2/33fbd431cb835ea285bd780e4abd2054dbdefc247c8332beb5912b05eb3c0a8e/diff
bd2fc135adc4 --> /var/lib/docker/overlay2/5e7f427d1c1e1490898306e77a576618321eff1a32ee33b473c28b07eaf4942b/diff
2ba38e44c757 --> /var/lib/docker/overlay2/9e71c8304cba18add3b6c02c06d491a0682d24cf44405c8051a0432a0dd0483f/diff
d6bd472a33fa --> /var/lib/docker/overlay2/4e7df6d99f783790c9050122690579d35412349df1a929efb022c3b7dae025cf/diff
c983af33248b --> /var/lib/docker/overlay2/03d5221208046c6d43059c517f7acc28ee16c5aadeb3058f7ed04ab505c2401e/diff
bb064b2d233b --> /var/lib/docker/overlay2/732353b15817a43781e0bf8b1f49caffe0292b2fd64fa928b802050ee1fa5c1c/diff
7d38dfa5e151 --> /var/lib/docker/overlay2/90d587f99d1046cd69fb8d9d9147dcd810d45b36d75c06b79e1384e33bcb7d1e/diff
f043dc759b60 --> /var/lib/docker/overlay2/c8be401e0f6ae8ed77f2cadafcac47a7675e6af969763680c83d19a9e442612b/diff
076b096a3668 --> /var/lib/docker/overlay2/7597a7d509ad516f54e932ce6b089b100f994c59cb0475f929d7c9f79c1d6094/diff
66c86152d41f --> /var/lib/docker/overlay2/b786a0628425df54d48b3c710497462260089614d5aacfd694e13c754558ac36/diff
d560ef4d46b9 --> /var/lib/docker/overlay2/99f7b1466a9d0ad93fe4271356521de89de11c6070382c3d92e46e1e11a0193b/diff
b9fc7d35ca7e --> /var/lib/docker/overlay2/79821b30325c9e2952d82f499cd668ef9df5e0be182c10a8fdacaba73c46d2bd/diff
6d5ff329ed79 --> /var/lib/docker/overlay2/42c6256f2df5af834f54d7bf61379020d0c728bebe5979d1d5f04c505ea0e65e/diff
b2804633193b --> /var/lib/docker/overlay2/f76b0824066e6b1eb91d3a472fca037fb2c61882e4f38875f69832f53fc32378/diff
0afa7d661aef --> /var/lib/docker/overlay2/adbba6a3bb0d666f3794f0f26150ac35d92100c7a65173085f531f2c0117ace7/diff
3c00d2fdd100 --> /var/lib/docker/overlay2/132cdee7473877731c842e3a01bd20a4f6de855ce172312af54e45533dee66b9/diff
e107928ad9fd --> /var/lib/docker/overlay2/6c9a6ab1746ed53c99b5ae1f362d81363f364a3d2e7d0c7df6fc36b94ab9ec66/diff
a470a999c743 --> /var/lib/docker/overlay2/c32d033b6eca29c97e1b1fcfa4957a4fad515e0d81be63722a7f32878aa726dc/diff
40bbbb63baf0 --> /var/lib/docker/overlay2/71a9da3ac3e379bdefd80d58feb72b2cb190034d25eeaffc9a55fdaf28fee9c3/diff
6ca8c68dec04 --> /var/lib/docker/overlay2/3bd0c6b04bccc56b5a7c365d4bbdb8f48cb1032a717522a66eecb4b121d268f8/diff
ffc312d5a21c --> /var/lib/docker/overlay2/a17a4e27d0d13b29063e340e801255e9b411560a6befc6530fa34d27387bd990/diff
edc074aded2a --> /var/lib/docker/overlay2/47f8b0cb6f7a9654e17ba867205765d05c7f801c7212d6444f9edec22b15227b/diff
51cc6f4e56ac --> /var/lib/docker/overlay2/c59a9826c86a3e88d04be09a58cf9a6fae01f186ae156fc3ce51cd715ddcbc3a/diff
f22418805d80 --> /var/lib/docker/overlay2/e3911f2e28ef13ac3e3b88d98e6f3d7a53cf1d67e0fcc1bd9fd386677e97bac3/diff
0b1d56931632 --> /var/lib/docker/overlay2/e826001071befa9e3e6d1eec0fbcbe915b09f361a4a8f83df6b7d43a763a45d8/diff
a5b7061fbb9c --> /var/lib/docker/overlay2/67ae7aba39c07636e4c33743008b1fe68a6f646884f9654f5b67bb1d3363a054/diff
ed831a901d9a --> /var/lib/docker/overlay2/8e8da59935d7f0c9baaa63e6bb56cc5e70115b4504aec7b11a022df04aee36f0/diff
2d680193ee1e --> /var/lib/docker/overlay2/2c77766057f10b25521d26b7738fb8f0c7b1fe2a92c353969c163265904d9da9/diff
```



根据匹配的ID，查看对应容器

```bash
97069ce8f167 --> /var/lib/docker/overlay2/d26846eba748521896fb80c5ef13150ebd1785878634c53d0ee7ed56d649ca3e/diff
```



```bash
# docker inspect 97069ce8f167|grep -i data
                "/data/api/logs:/logs:rw"
        
                
# cd  /data/api/

# 重建释放空间
# docker-compose down;docker-compose up -d --build

                
                
```

