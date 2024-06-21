```shell
def jobName = "item_name"
def maxNumber = 100
Jenkins.instance.getItemByFullName(jobName).builds.findAll {
    it.number <= maxNumber
}.each {
    it.delete()
}
```