## 添加依赖

```
compile 'com.android.support:recyclerview-v7:23.2.0'
```

## Adapter变化

既然要有**ListView**与**GridView**的效果自然不能少了**adapter**,而**RecyclerView**的**adapter**主要有两个重大的变化，原来**adapter**中的**getView()**方法取消了，替换成了**onCreateViewHolder()**与**onBindViewHolder()**。其实就是分成了两部分，一部分创建视图另一部分绑定数据，是不是感觉层次更清晰了呢，下面主要介绍之两大方法的使用。
### onCreateViewHolder
说白了这个方法就是创建视图，在创建之前要创建个**ViewHolder**相信在原来的**adapter**中应该有使用过吧.这里继承**RecyclerView.ViewHolder**

```
public static class NormalViewHolder extends RecyclerView.ViewHolder {
        @Bind(R.id.item_tv)
        TextView itemTv;
        public NormalViewHolder(View itemView) {
            super(itemView);
            ButterKnife.bind(this, itemView);
        }
    }   
```

然后在**onCreateViewHolder**中调用：

```
	@Override
    public NormalViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        return new NormalViewHolder(mLayoutInflater.inflate(R.layout.item_text, parent, false));
    }
    
```
### onBindViewHolder
视图创建完毕就是简单的绑定了，使用holder对控件进行绑定设置数据：

```
    @Override
    public void onBindViewHolder(NormalViewHolder holder, int position) {
        holder.itemTv.setText((CharSequence) mListData.get(position));
    }
```

### setLayoutManager

```
switch (type) {
            case App.LINEAR_LAYOUT:
            	//ListView
                recyclerView.setLayoutManager(new LinearLayoutManager(getActivity()));
                break;
            case App.GRID_LAYOUT:
            	//GridView
                gridLayoutManager = new GridLayoutManager(getActivity(),3);
                recyclerView.setLayoutManager(gridLayoutManager);
                break;                
            case App.STAGGERED_GRID_LAYOUT:
            	//Can customize the waterfall flow
                recyclerView.setLayoutManager(new StaggeredGridLayoutManager(2, OrientationHelper.VERTICAL));
                break;
            default:
                break;
        }
```

## ListView与GridView嵌套效果
如果要实现首尾是**ListView**布局效果，中间的**GridView**效果，我们可以使用**RecyclerView.Adapter**提供的**getItemViewType(int position)**方法，注意这里与**ListView**的**adapter**不同只有这一个方法，没有**getViewTypeCount() **方法，我们可以通过首尾要显示的行数来控制显示的布局

```
@Override
    public int getItemViewType(int position) {
        mHeadCount = getHeadCount();
        mContentCount = getContentCount();
        mBottomCount = mHeadCount + mContentCount;
        if (mHeadCount > position) {
            return App.LINEAR_LAYOUT;//ListView
        } else if (mBottomCount <= position) {
            return App.STAGGERED_GRID_LAYOUT;//Can customize the waterfall flow
        } else {
            return App.GRID_LAYOUT;//GridView
        }
    }

```

>*根据不同的viewtype返回不同的布局*


```
@Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        switch (viewType) {
            case App.LINEAR_LAYOUT:
                return onCreateHeadViewHolder(parent);
            case App.GRID_LAYOUT:
                return onCreateContentViewHolder(parent);
            case App.STAGGERED_GRID_LAYOUT:
                return onCreateBottomViewHolder(parent);
            default:
                return null;
        }
    }
```

## RecyclerView动画
**RecyclerView**的**adapter**提供了一些简单的动画实现，下面是一部分:

* notifyItemRemoved(int position)
* notifyItemInserted(int position)
* notifyItemChanged(int position)
* notifyItemRangeChanged(int positionStart,int itemCount)

只要在数据改变时通过**adapter**通知就行：

```
 recyclerView.addOnItemTouchListener(new OnItemTouchListener(recyclerView) {
             @Override
             public void onItemClick(RecyclerView.ViewHolder vh) {
                 if (vh.getLayoutPosition() != 1) {
                     adapter.mListData.add("add" + vh.getLayoutPosition());
                     adapter.notifyItemInserted(vh.getLayoutPosition());
                 } else {
                     adapter.mListData.remove(vh.getLayoutPosition());
                     adapter.notifyItemRemoved(vh.getLayoutPosition());
                 }
             }
         });
```

使用**recyclerView**的**setItemAnimator()**可以设置动画：

```
recyclerView.setItemAnimator(new DefaultItemAnimator());
```

当然你也可以实现自己的动画，只要**extends** ItemAnimator重写几个方法，这里就不展开了，读者可以自己去试试。


### RecyclerBaseCursorAdapter
这个抽象类大部分都是与**ListView**的**CursorAdapter**相同，只是有几个地方需要特别提醒下

* 原来的有notifyDataSetInvalidated()方法，但RecyclerView没有提供该方法，我们可以使用notifyDataSetChanged()替代
* 原来的hasStableIds() 方法也没用，我们也可以使用setHasStableIds(true)设置为ture来实现相同的效果
* 原来的**getView()**方法要替换成**onBindViewHolder()**方法

# 效果图

![效果图](https://github.com/idisfkj/RecyclerView/raw/master/image/RecyclerView.gif)

![效果图](https://github.com/idisfkj/RecyclerView/raw/master/image/RecyclerView1.gif)

# 结语

跟多详解：
[RecyclerView深入浅出](http://idisfkj.github.io/2016/04/02/RecyclerView%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BA/)

[RecyclerView item点击你真的会么](http://idisfkj.github.io/2016/05/22/RecyclerView-item%E7%82%B9%E5%87%BB%E4%BD%A0%E7%9C%9F%E7%9A%84%E4%BC%9A%E4%B9%88/)

[RecyclerView的拖动、滑动删除](http://idisfkj.github.io/2016/06/04/RecyclerView%E7%9A%84%E6%8B%96%E5%8A%A8%E3%80%81%E6%BB%91%E5%8A%A8%E5%88%A0%E9%99%A4/)