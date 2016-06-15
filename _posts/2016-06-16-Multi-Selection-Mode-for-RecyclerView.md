---
layout: post
title:  "Multi-Selection Mode for RecyclerView"
date: 2016-06-14
comments: true
categories: [Android, RecyclerView]
---

### Overview

Not like ListView, RecyclerView doesn't provide any tool for item selection. So we are on our own.

Since `RecyclerView` doesn't care about the visual representation of individual items, extending `RecyclerView` could be ruled out. `RecyclerView.Adapter` possesses all the information about the data set in this `RecyclerView`, also creates/binds views for each item. Thus extending `RecyclerView.Adapter` here to handle selecting state.

### Methods for selected state

Based on the use case, add the following methods into `RecyclerView.Adapter` subclass:

* `boolean isSelected(int position)`
* `void switchSelectedState(int position)`
* `void clearSelectedState()`
* `int getSelectedItemCount()`
* `List<Integer> getSelectedItems()`

Code :

```java
public class MultiSelectRecyclerViewAdapter extends RecyclerView.Adapter {
    private SparseBooleanArray mSelectedItems;
    private boolean mIsInChoiceMode;
    private PhotoGridViewHolder.ClickListener mClickListener;
    
    public boolean isSelected(int position) {
    	return getSelectedItems().contains(position);
    }

    public void switchSelectedState(int position) {
        if (mSelectedItems.get(position)) {       // item has been selected, de-select it.
            mSelectedItems.delete(position);
        } else {
            mSelectedItems.put(position, true);
        }
        notifyItemChanged(position);
    }

    public void clearSelectedState() {
        List<Integer> selection = getSelectedItems();
        mSelectedItems.clear();
        for (Integer i : selection) {
            notifyItemChanged(position);
        }
    }

    public int getSelectedItemCount() {
        return mSelectedItems.size();
    }

    public List<Integer> getSelectedItems() {
        List<Integer> items = new ArrayList<>(mSelectedItems.size());
        for (int i = 0; i < mSelectedItems.size(); ++i) {
            items.add(mSelectedItems.keyAt(i));
        }
        return items;
    }
    
    public void setIsInChoiceMode(boolean isInChoiceMode) {
        this.mIsInChoiceMode = isInChoiceMode;
    }

    public boolean getIsInChoiceMode() {
        return mIsInChoiceMode;
    }
}
```

To update item's visual based on selected status, override onBindViewHolder:

```java
public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
    PhotoGridViewHolder viewHolder = (PhotoGridViewHolder) holder;

    // ...

    if (mIsInChoiceMode) {
        viewHolder.checkbox.setVisibility(View.VISIBLE);
        viewHolder.checkbox.setChecked(mSelectedItems.get(position));
    } else {
        viewHolder.checkbox.setChecked(false);      // clear all checkbox if we want.
        viewHolder.checkbox.setVisibility(View.INVISIBLE);
    }
}
```

### Item ClickListener interface
In `ViewHolder`, define an interface for **click**/**long click** handling:

```java
public static class PhotoGridViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener, View.OnLongClickListener {

    // ...

    @Override
    public void onClick(View v) {
        if (listener != null) {
            listener.onItemClicked(SectionedGridRecyclerViewAdapter.sectionedPositionToPosition(getAdapterPosition()));
        }
    }

    @Override
    public boolean onLongClick(View view) {
        return listener != null
                && listener.onItemLongClicked(SectionedGridRecyclerViewAdapter.sectionedPositionToPosition(getAdapterPosition()));
    }

    public interface ClickListener {
        void onItemClicked(int position);
        boolean onItemLongClicked(int position);
    }
}
```

In the `RecyclerView`, implement this interface to handle:

* enter/exit selection mode
* select/de-select item(s)

```java
mAdapter = new MultiSelectRecyclerViewAdapter(getContext(), new MultiSelectRecyclerViewAdapter.PhotoGridViewHolder.ClickListener() {
	@Override
	public void onItemClicked(int position) {
	    Log.d(TAG, "onItemClicked, position: " + position);
	    if (mAdapter.getIsInChoiceMode()) {
	        mAdapter.switchSelectedState(position);
	    } else {
	        // normal click handling
            // ...
	    }
	}

	@Override
	public boolean onItemLongClicked(int position) {
	    mAdapter.beginChoiceMode(position);
	    return true;
	}
});

setAdapter(mAdapter);
```

### Update toolbar based on selected status

I want the toolbar's subtitle show the amount of item selected. This feature is implemented with RxBus (Refer to [this](http://www.jianshu.com/p/ca090f6e2fe2/) or [this](http://nerds.weddingpartyapp.com/tech/2014/12/24/implementing-an-event-bus-with-rxjava-rxbus/)).

In the fragment/activity, subscribe the toolbar updating event:

```java
mUpdateToolbarSubscription = RxBus.getInstance().toObserverable(UpdateToolbarEvent.class)
                .subscribe(new Action1<UpdateToolbarEvent>() {
                    @Override
                    public void call(UpdateToolbarEvent event) {
                        updateToolbar(event);
                    }
                });
                
private void updateToolbar(UpdateToolbarEvent event) {
    Log.d(TAG, "update title");
    String title = event.getTitleContent();
    String subtitle = event.getSubtitleContent();

    switch (event.getToolbarMode()) {
        case UpdateToolbarEvent.NORMAL_MODE:      // normal mode: only a main title.
            Log.d(TAG, "Update toolbar event: NORMAL_MODE");
            setNavigationIcon();
            mIsInChoiceMode = false;
            setTitles(title, null);
            showChoiceModeActions(false);
            break;

        case UpdateToolbarEvent.MULTI_SELECT_MODE:
            Log.d(TAG, "Update toolbar event: MULTI_SELECT_MODE");
            materialMenu.animateIconState(MaterialMenuDrawable.IconState.X);
            mIsInChoiceMode = true;
            setTitles(title, subtitle);
            showChoiceModeActions(true);
    }
}

private void setTitles(String title, String subtitle) {
    // set title if needed
    if (title != null && title.length() > 0) {
        mToolbar.setTitle(title);
    }
    // set subtitle
    mToolbar.setSubtitle(subtitle);     // if no subtitle needed, setSubtitle(null)
}
```

Then in the `MultiSelectRecyclerViewAdapter`, post an event every time selected status changed.

```java
public void switchSelectedState(int position) {
    // ...
    RxBus.getInstance().post(new UpdateToolbarEvent(UpdateToolbarEvent.MULTI_SELECT_MODE, null, getSelectedItemCount() + " Selected"));
}
    
public void clearSelectedState() {
    // ...
    RxBus.getInstance().post(new UpdateToolbarEvent(UpdateToolbarEvent.NORMAL_MODE, null, getSelectedItemCount() + " Selected"));
}
```