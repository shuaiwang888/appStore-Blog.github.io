### 0 由来

今天在开发中发现学习到了一点，如果在方法A的逻辑代码处理后，需要对同一个数据库表多次操作，或者对不同数据库表操作需要添加事务，但是为了代码的完备性，需要单独把最后的数据库操作代码拿出来放到方法B中。所以产生了在service层需要使用非事务方法A调用事务方法B。

### 1 简介

查资料可知，在springAOP的用法中，只有代理的类才会被切入，我们在controller层调用service的方法的时候，是可以被切入的（**所以常常我们在controller中在接口上添加校验权限注解就是利用AOP的切入的！**），但是如果我们在service层A方法中，调用B方法，切点切的是B方法，那么这时候是不会切入的，解决办法就是如上所示，在A方法中使用((Service)AopContext.currentProxy()).B() 来调用B方法，这样一来，就能切入了！

B方法被A调用，对B方法的切入失效，但加上AopContext.currentProxy()创建了代理类，在代理类中调用该方法前后进行切入。对于B方法proxyA中调用只能对A进行增强，A里面调用B使用的是对象.B(),而不是$proxy.B(),所以对B的切入无效。

**`AopContext.currentProxy()`使用了`ThreadLocal`保存了代理对象，因此`AopContext.currentProxy().B()`就能解决。**

### 2 AopContext.currentProxy()该用法的意义

具体的用法在下面的代码中可以体会(部分代码)：

```java
@Service
@Slf4j
public class GardenManageV2ServiceImpl implements IGardenManageV2Service {

    @Autowired
    private IGardenAction gardenAction;

    // 方法A1
    private void doDownMovementSortForGarden(GardenEntity currentNodeIdEntity, GardenEntity targetNodeIdEntity) {
        log.info("进行节点上移，选定列：{}, 目标列：{}", currentNodeIdEntity, targetNodeIdEntity);
        List<GardenEntity> gardenEntityList = gardenAction.lambdaQuery()
                .ge(GardenEntity::getOrderId, currentNodeIdEntity.getOrderId())
                .le(GardenEntity::getOrderId, targetNodeIdEntity.getOrderId())
                .list();
        List<GardenEntity> updateGardenListTemp = new ArrayList<>();
        List<GardenEntity> updateGardenList = new ArrayList<>();
        for (GardenEntity gardenEntity : gardenEntityList) {
            GardenEntity updateGardenTemp = new GardenEntity();
            GardenEntity updateGarden = new GardenEntity();
            if (gardenEntity.getId().equals(currentNodeIdEntity.getId())) {
                updateGardenTemp.setId(gardenEntity.getId());
                updateGardenTemp.setOrderId(Math.negateExact(targetNodeIdEntity.getOrderId()));
                updateGarden.setId(gardenEntity.getId());
                updateGarden.setOrderId(targetNodeIdEntity.getOrderId());
                updateGardenListTemp.add(updateGardenTemp);
                updateGardenList.add(updateGarden);
            } else {
                updateGardenTemp.setId(gardenEntity.getId());
                updateGardenTemp.setOrderId(Math.negateExact(gardenEntity.getOrderId() - 1));
                updateGarden.setId(gardenEntity.getId());
                updateGarden.setOrderId(gardenEntity.getOrderId() - 1);
                updateGardenListTemp.add(updateGardenTemp);
                updateGardenList.add(updateGarden);
            }
        }

        try {
            ((GardenManageV2ServiceImpl) AopContext.currentProxy()).doSortGardenForMYSQL(updateGardenListTemp, updateGardenList);
        } catch (Exception e) {
            throw new BusinessException(SORT_FAIL + e);
        }
    }

    // 方法B
    @Transactional(rollbackFor = Throwable.class)
    public void doSortGardenForMYSQL(List<GardenEntity> updateGardenListTemp, List<GardenEntity> updateGardenList) {
        gardenAction.saveOrUpdateBatch(updateGardenListTemp);
        gardenAction.saveOrUpdateBatch(updateGardenList);
    }

    // 方法A2
    private void doUpMovementSortForGarden(GardenEntity currentNodeIdEntity, GardenEntity targetNodeIdEntity) {
        log.info("进行节点下移，选定列：{}, 目标列：{}", currentNodeIdEntity, targetNodeIdEntity);
        List<GardenEntity> buildingEntityList = gardenAction.lambdaQuery()
                .ge(GardenEntity::getOrderId, targetNodeIdEntity.getOrderId())
                .le(GardenEntity::getOrderId, currentNodeIdEntity.getOrderId())
                .list();
        List<GardenEntity> updateGardenListTemp = new ArrayList<>();
        List<GardenEntity> updateGardenList = new ArrayList<>();
        for (GardenEntity gardenEntity : buildingEntityList) {
            GardenEntity updateGardenTemp = new GardenEntity();
            GardenEntity updateGarden = new GardenEntity();
            if (gardenEntity.getId().equals(currentNodeIdEntity.getId())) {
                updateGardenTemp.setId(gardenEntity.getId());
                updateGardenTemp.setOrderId(Math.negateExact(targetNodeIdEntity.getOrderId()));
                updateGarden.setId(gardenEntity.getId());
                updateGarden.setOrderId(targetNodeIdEntity.getOrderId());
                updateGardenListTemp.add(updateGardenTemp);
                updateGardenList.add(updateGarden);
            } else {
                updateGardenTemp.setId(gardenEntity.getId());
                updateGardenTemp.setOrderId(Math.negateExact(gardenEntity.getOrderId() + 1));
                updateGarden.setId(gardenEntity.getId());
                updateGarden.setOrderId(gardenEntity.getOrderId() + 1);
                updateGardenListTemp.add(updateGardenTemp);
                updateGardenList.add(updateGarden);
            }
        }
        try {
            ((GardenManageV2ServiceImpl) AopContext.currentProxy()).doSortGardenForMYSQL(updateGardenListTemp, updateGardenList);
        } catch (Exception e) {
            throw new BusinessException(SORT_FAIL + e);
        }
    }
}
```

## 3 总结
>**在不同类中，非事务方法A调用事务方法B，事务生效。
在同一个类中，事务方法A调用非事务方法B，事务具有传播性，事务生效
在不同类中，事务方法A调用非事务方法B，事务生效。**
