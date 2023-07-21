# 线性表的顺序存储—顺序表

线性表的顺序存储—顺序表
<!--more-->

线性表的顺序存储结构：把线性表中的所有元素按照其逻辑顺序依次存储到从计算机存储器中指定存储位置开始的一块连续的存储空间中。 
这样,线性表中第一个元素的存储位置就是指定的存储位置，第i+1个元素（1≤i≤n-1）的存储位置紧接在第i个元素的存储位置的后面。 
说明：由于C中数组的下标从0开始，线性表的第i个元素ai存放顺序表的第i-1位置上。为了清楚，将ai在逻辑序列中的位置称为逻辑位序，在顺序表中的位置称为物理位序。 
线性表<----> 逻辑结构 
顺序表 <---> 存储结构 
 
优点： 
无须为表示表中元素之间的逻辑关系而增加额外的存储空间。 
可以快速地存取表中任意位置的元素。 
 
缺点： 
插入和删除操作需要移动大量元素。 
当线性表长度变化较大时，难以确定存储空间的容量。 
容易造成存储空间的“碎片”。 
 
 
#define LIST_INIT_SIZE 100  // 线性表存储空间的初始分配量
#define LISTINCREMENT 10    // 线性表存储空间的分配增量
#define OK 0
#define ERROR 1
#define OVERFLOW 0
typedef int ElemType;

/** 线性表的动态分配顺序存储结构 */
typedef struct
{
    ElemType *elem;  // 存储空间基址
    int length;  // 当前长度
    int listsize;  //当前分配的存储容量（以sizeof(ElemType)为单位）
}SqList;

/*************************************************
 * Function: InitList
 * Description: 初始化空间
 * param: L SqList* 顺序表
 * Return: int
 * Others: 时间复杂度为O(1)
 *************************************************/
int InitList(SqList *L)
{
    L->elem = (ElemType *)malloc(LIST_INIT_SIZE * sizeof(ElemType));
    if(!L->elem)
        exit(OVERFLOW);  //存储分配失败
    L->length = 0;  // 空表长度为0
    L->listsize = LIST_INIT_SIZE;  // 初始存储容量
    return OK;
}
/*************************************************
 * Function: AddListSize
 * Description: 为顺序表增加空间
 * param: L SqList* 顺序表
 * Return: int
 * Others: 时间复杂度为O(1)
 *************************************************/
int AddListSize(SqList *L)
{
    ElemType *newbase = (ElemType *)realloc(L->elem,(L->listsize + LISTINCREMENT) * sizeof(ElemType));
    if(!newbase)
    {
        exit(OVERFLOW);
    }
    L->elem = newbase;
    L->listsize += LISTINCREMENT;
    return OK;
}
/*************************************************
 * Function: CreateList
 * Description: 为顺序表赋值
 * param: L SqList* 顺序表
          a[] ElemType 数组
          n int 长度
 * Others: 时间复杂度为O(n)
 *************************************************/
void CreateList(SqList *L, ElemType a[], int n)
{
    int i;

    /** 如果顺序表没有容量，初始化容量 */
    if(L->listsize == 0)
    {
        InitList(L);  //调用初始化函数
    }

    /** 如果顺序表的容量小于长度，增加容量 */
    if(L->listsize < n)
    {
        AddListSize(L);  // 调用增加容量函数
    }

    /** 顺序表赋值 */
    for(i = 0; i < n; i++)
    {
        L->elem[i] = a[i];
    }
    L->length = n;  // 线性表长度赋值
}

 /*************************************************
 * Function: ListInSert
 * Description: 在顺序表中第i个位置前插入新元素e，
 * param: L SqList* 顺序表
 *        i int 插入的位置
 *        e ElemType 插入的元素
 * Return: int
 * Others: 时间复杂度为O(n)
 *************************************************/
int ListInSert(SqList *L, int i, ElemType e)
{
    int k;

    /** 如果顺序表的长度大于或等于现象表的容量，增加容量 */
    if(L->length >= L->listsize)
    {
        AddListSize(L);  // 调用增加容量函数
    }

    /** 位置应该从1开始到表长+1，位置i如果小于1或者大于顺序表的长度，函数结束 */
    if(i < 1 || i > L->length+1)
    {
        return ERROR;
    }

    /** 插入位置以及之后的元素后移 */
    for(k = L->length-1; k >= i-1; k--)  //
    {
        L->elem[k+1] = L->elem[k];
    }
    L->elem[i-1] = e;  // 插入元素
    L->length +=1;  //表长加1
    return OK;
}

/*************************************************
 * Function: ListDelete
 * Description: 删除顺序表中第i个位置的元素并将删除的元素赋值给e
 * param: L SqList* 顺序表
 *        i int 删除的位置
 *        e ElemType 删除的元素
 * Return: int
 * Others: 时间复杂度为O(n)
 *************************************************/
int ListDelete(SqList *L, int i, ElemType *e)
{
    int k;

    /** 如果顺序表的长度为零，顺序表中没有元素，函数结束 */
    if(L->length == 0)
    {
        return ERROR;
    }

    /** 位置应该从1开始到表长+1，位置i如果小于1或者大于顺序表的长度，函数结束 */
    if(i < 1 || i > L->length)
    {
        return ERROR;
    }

    *e = L->elem[i-1];  // 将删除的元素给e

    /** 删除位置之后的元素前移 */
    for(k = i; k < L->length; k++)
    {
        L->elem[k-1] = L->elem[k];
    }
    L->length--;  // 线性表的长度-1
    return OK;
}

/*************************************************
 * Function: DestroyList
 * Description: 删除顺序表,释放存储空间
 * param: L SqList* 顺序表
 * Return: int
 * Others: 时间复杂度为O(1)
 *************************************************/
void DestroyList(SqList *L)
{
    L->length = 0; // 长度0
    L->listsize = 0; // 容量0
    free(L);  // 释放空间
}

/*************************************************
 * Function: ClearList
 * Description: 清空顺序表中的元素
 * param: L SqList* 顺序表
 * Return: int
 * Others: 时间复杂度为O(1)
 *************************************************/
int ClearList(SqList *L)
{
    L->length = 0;  // 线性表的长度设置为0
    return OK;
}

/*************************************************
 * Function: ListEmpty
 * Description: 顺序表是否为空
 * param: L SqList* 顺序表
 * Return: int 0代表顺序表为空，1代表顺序表不为空
 * Others: 时间复杂度为O(1)
 *************************************************/
int ListEmpty(SqList *L)
{
    if(L->length >0)
    {
        return ERROR;
    }
    else
    {
        return OK;
    }
}

/*************************************************
 * Function: ListLength
 * Description: 顺序表的长度
 * param: L SqList* 顺序表
 * Return: int 顺序表的长度
 * Others: 时间复杂度为O(1)
 *************************************************/
int ListLength(SqList *L)
{
    return L->length;
}

/*************************************************
 * Function: GetElem
 * Description: 获取顺序表中第i个位置的元素，并赋值给e
 * param: L SqList* 顺序表
 *        i int 元素的位置
 *        e ElemType 元素
 * Return: int
 * Others: 时间复杂度为O(1)
 *************************************************/
int GetElem(SqList *L,int i, ElemType *e)
{
    /** 位置应该从1开始到表长+1，位置i如果小于1或者大于顺序表的长度，函数结束 */
    if(i <1 || i > L->length)
    {
        return ERROR;
    }
    *e = L->elem[i-1];  // 逻辑位置i 物理位置应为i-1
    return OK;
}

/*************************************************
 * Function: LocateElem
 * Description: 获取顺序表中元素所在的位置
 * param: L SqList* 顺序表
 *        e ElemType 元素
 * Return: int 元素所在的位置
 * Others: 时间复杂度为O(n)
 *************************************************/
int LocateElem(SqList *L, ElemType e)
{
    int i = 0;

    /** 循环顺序表的长度，找出元素。 */
    while(i < L->length && L->elem[i] != e)
    {
        i++;
    }

    /** 循环结束后，i大于或等于顺序表的长度，说明没有找到，函数结束 */
    if(i >= L->length)
    {
        return ERROR;
    }

    return i+1;  // 逻辑位置i从1开始，物理位置从0开始，返回逻辑位置。
}

/*************************************************
 * Function: DispList
 * Description: 输出显示显示顺序表
 * param: L SqList* 顺序表
 * Others: 时间复杂度为O(n)
 *************************************************/
void DispList(SqList *L)
{
    int i;

    /** 如果顺序表的长度为零，顺序表中没有元素，函数结束 */
    if(L->length == 0)
    {
        return ERROR;
    }

    /** 循环顺序表，输出元素。 */
    for(i = 0; i < L->length; i++)
    {
        printf("%d,",L->elem[i]);
    }
    printf("\n");
}

/*************************************************
 * Function: unionList
 * Description: 合并两个顺序表，将说有在顺序表LB中但不在LA中的元素插入到LA中
 * param: LA SqList* 顺序表
 * param: LB SqList* 顺序表
 * Others: 时间复杂度为O(ListLength(LA)*ListLength(LB))
 *************************************************/
void unionList(SqList *LA, SqList *LB)
{
    int lalen = LA->length;  // 获得顺序表LA的长度
    int lblen = LB->length;  // 获取顺序表LB的长度
    int i;
    ElemType e;

    /** 循环顺序表LB 找出在LB中但不在LA中的元素插入到LA中 */
    for(i = 1; i <= lblen; i++)
    {
        GetElem(LB,i,e);  // 调用GetElem函数，取LB中第i个数据元素赋给e

        /** LA中不存在和e相同者,插入到LA中 */
        if(!LocateElem(LA,e))
        {
            ListInSert(LA,++lalen,e);
        }
    }
}

/*************************************************
 * Function: Inverse
 * Description: 顺序表的逆置
 * param: L SqList* 顺序表
 * Others: 时间复杂度为O(n)
 *************************************************/
void Inverse(SqList *L)
{
    int low=0,high=L->length-1;
    ElemType temp;
    int i;
    for(i=0; i<L->length/2; i++)
    {
        temp=L->elem[low];
        L->elem[low++]=L->elem[high];
        L->elem[high--]=temp;
    }
}
int main()
{
    SqList L;
    int length,a,i,pos;
    ElemType temp;
    InitList(&L);
    printf("请先初始化顺序表\n");
    printf("输入顺序表的长度:");
    scanf("%d",&length);
    printf("输入顺序表的元素:");
    for(i=0; i<length; i++)
    {
        scanf("%d",&temp);
        ListInSert(&L,i+1,temp);
    }
    printf("创建好的顺序表L=");
    DispList(&L);
    while(1)
    {
        printf("功能：\n");
        printf("\t【1】显示顺序表元素\n\t【2】插入元素\n\t【3】删除元素\n\t【4】查找元素\n\t【5】显示顺序表长度\n\t【6】倒置顺序表\n\t【0】退出\n");
        printf("请选择功能：");
        scanf("%d",&a);
        if(a == 0)
        {
            return 0;
        }
        else if(a == 1)
        {
            printf("创建好的顺序表La=");
            DispList(&L);
        }
        else if(a == 2)
        {
            printf("请输入插入位置：");
            scanf("%d",&pos);
            printf("请输入插入元素：");
            scanf("%d",&temp);
            int is = ListInSert(&L,pos,temp);
            if( is == 0)
            {
                printf("插入成功！\n");
            }
            else
            {
                printf("插入失败！\n");
            }
        }
        else if(a == 3)
        {
             printf("请输入删除元素的位置：");
             scanf("%d",&pos);
             int is = ListDelete(&L, pos, &temp);
             if(is == 0)
             {
                 printf("删除的元素为%d！\n",temp);
             }
             else
             {
                 printf("删除失败！\n");
             }
        }
        else if(a == 4)
        {
            printf("请输入要查找的元素：");
            scanf("%d",&temp);
            printf("要查找的元素在：%d\n",LocateElem(&L,temp));
        }
        else if(a == 5)
        {
            printf("顺序表的长度为：%d\n",ListLength(&L));
        }
        else if(a == 6)
        {
            Inverse(&L);
            printf("倒置后的顺序表L=");
            DispList(&L);
        }
    }
    return 0;
} 
 




---

> 作者: wang  
> URL: https://codingroam.github.io/post/%E7%BA%BF%E6%80%A7%E8%A1%A8%E7%9A%84%E9%A1%BA%E5%BA%8F%E5%AD%98%E5%82%A8%E9%A1%BA%E5%BA%8F%E8%A1%A8/  

