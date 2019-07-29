

一。减少View.inflate的调用 必须重用视图回收器返回缓存视图

二。减少低性能查询findViewById(树节点的遍历) 必须使用集合ViewHolder缓存初始化的视图控件

优化一：
LayoutInflater.from(mContext).inflate(R.layout.view_select_contact_item, null);是非常耗时的操作，每次调用都会加载布局文件R.layout.view_select_contact_item；

在getView()方法中复用convertView，只在初始化时使用LayoutInflater，效率提高很多。

```java
   @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        if (convertView == null) {
            convertView = LayoutInflater.from(mContext).inflate(R.layout.view_select_contact_item, null);
        }

        //初始化
        ImageView ivPhoto = (ImageView) convertView.findViewById(R.id.select_contact_item_iv_photo);
        TextView tvName = (TextView) convertView.findViewById(R.id.select_contact_item_tv_name);
        TextView tvNumber = (TextView) convertView.findViewById(R.id.select_contact_item_tv_number);

        //绑定数据
        ContactInfo contactInfo = mData.get(position);
        tvName.setText(contactInfo.name);
        tvNumber.setText(contactInfo.number);
        //联系人头像
        Bitmap bitmap = ContactUtils.getContactPhoto(mContext, contactInfo.contact_Id);
        if (bitmap == null) {
            ivPhoto.setImageResource(R.drawable.ic_contact);
        }else {
            ivPhoto.setImageBitmap(bitmap);
        }
        return convertView;
    }
```


优化二：
convertView.findViewById(R.id.select_contact_item_iv_photo);等通过id查找控件，会从布局文件中一个标签一个标签的找，也是降低效率的操作。


该优化效率提高一点。
​

```java
static class ViewHolder {
        ImageView ivPhoto;
        TextView tvName;
        TextView tvNumber;
    }
​
​
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder;
​
​
        if (convertView == null) {
            convertView = LayoutInflater.from(mContext).inflate(R.layout.view_select_contact_item, null);
​
​
            holder = new ViewHolder();
            //初始化
            holder.ivPhoto = (ImageView) convertView.findViewById(R.id.select_contact_item_iv_photo);
            holder.tvName = (TextView) convertView.findViewById(R.id.select_contact_item_tv_name);
            holder.tvNumber = (TextView) convertView.findViewById(R.id.select_contact_item_tv_number);
​
​
            convertView.setTag(holder);
        }else {
            holder = (ViewHolder) convertView.getTag();
        }
        //绑定数据
        ContactInfo contactInfo = mData.get(position);
        holder.tvName.setText(contactInfo.name);
        holder.tvNumber.setText(contactInfo.number);
        //联系人头像
        Bitmap bitmap = ContactUtils.getContactPhoto(mContext, contactInfo.contact_Id);
        if (bitmap == null) {
            holder.ivPhoto.setImageResource(R.drawable.ic_contact);
        }else {
            holder.ivPhoto.setImageBitmap(bitmap);
        }
        return convertView;
    }
```
