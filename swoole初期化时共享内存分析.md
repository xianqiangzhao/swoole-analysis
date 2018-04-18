# 共享内存的初期化在 \swoole-src-2.1.1\swoole.c  的swoole_init()中
```
\swoole-src-2.1.1\src\core\base.c

void swoole_init(void)
{
...
    //init global shared memory
    SwooleG.memory_pool = swMemoryGlobal_new(SW_GLOBAL_MEMORY_PAGESIZE, 1); // 全局共享内存分配
    if (SwooleG.memory_pool == NULL)
    {
        printf("[Master] Fatal Error: global memory allocation failure.");
        exit(1);
    }
    SwooleGS = SwooleG.memory_pool->alloc(SwooleG.memory_pool, sizeof(swServerGS)); //在共享内存中切出一块给SwooleGS用
    if (SwooleGS == NULL)
    {
        printf("[Master] Fatal Error: failed to allocate memory for SwooleGS.");
        exit(2);
    }

 ...
}
```
swMemoryGlobal_new 在\swoole-src-2.1.1\src\memory\MemoryGlobal.c 中，具体就内容就不展开了。
# 总之分配完的内存结构如下图所示。
![image.png](https://upload-images.jianshu.io/upload_images/9076770-d6dcee4d7e40f560.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 分配完共享内存 pool后，swoole_init函数中再分配一个SwooleGS结构体
SwooleGS = SwooleG.memory_pool->alloc(SwooleG.memory_pool, sizeof(swServerGS)); 
alloc实际执行的是
\swoole-src-2.1.1\src\memory\MemoryGlobal.c中的swMemoryGlobal_alloc函数
```
\swoole-src-2.1.1\src\memory\MemoryGlobal.c
static void *swMemoryGlobal_alloc(swMemoryPool *pool, uint32_t size)
{
    swMemoryGlobal *gm = pool->object;     //取出 swMemoryGlobal 对象 实际就是上图的addr =  0x7fffeb7b4068  =  page->memory
    gm->lock.lock(&gm->lock);     //加锁，防止多线程进程竞争进入临界区
    if (size > gm->pagesize - sizeof(swMemoryGlobal_page))  //判断分配的size是否 过大
    {
        swWarn("failed to alloc %d bytes, exceed the maximum size[%d].", size, gm->pagesize - (int) sizeof(swMemoryGlobal_page));
        gm->lock.unlock(&gm->lock);
        return NULL;
    }
    if (gm->current_offset + size > gm->pagesize - sizeof(swMemoryGlobal_page))  //判断共享内存中是否满足分配的size 
    {
        swMemoryGlobal_page *page = swMemoryGlobal_new_page(gm); //不满足的话再次申请共享内存
        if (page == NULL)
        {
            swWarn("swMemoryGlobal_alloc alloc memory error.");
            gm->lock.unlock(&gm->lock);
            return NULL;
        }
        gm->current_page = page;
    }
    void *mem = gm->current_page->memory + gm->current_offset;
    gm->current_offset += size;
    gm->lock.unlock(&gm->lock);
    return mem;
}

```
# 个人感觉共享内存的判断有点问题(也许我不对～～)
gm->pagesize - sizeof(swMemoryGlobal_page)
 应该是gm->pagesize - sizeof(swMemoryGlobal) -  sizeof(swMemoryPool)
 = gm->pagesize - gm->current_offset


