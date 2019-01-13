## v1とv2の設定は混在OK

```
Vagrant::Config.run do |config|
  # v1の設定
end

Vagrant.configure("2") do |config|
  # v2の設定
end
```
