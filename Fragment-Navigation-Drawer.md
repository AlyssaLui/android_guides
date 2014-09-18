In [[Common Navigation Paradigms]] cliffnotes, we discuss the various navigational structures available within Android applications. One of the most flexible is the [Navigation Drawer](http://developer.android.com/training/implementing-navigation/nav-drawer.html). The fully custom navigation drawer is totally managed by the user and can be read about on the [creating the navigation drawer](http://developer.android.com/training/implementing-navigation/nav-drawer.html#top) docs.

![Navigation Drawer](http://i.imgur.com/GG6JYZC.png)

## Usage

This guide instead explains how to setup a basic drawer filled with navigation items that switch different fragments into the content area. In this way, you can define multiple fragments, and then define the list of options which will display in the drawers items list. Each item when clicked will switch the relevant fragment into the activity's container view.

### Download Assets

Next, be sure to [download the drawer image assets](http://developer.android.com/downloads/design/Android_Navigation_Drawer_Icon_20130516.zip) necessary and add the images into each of your drawable folders.

### Android Support v4 JAR file

Verify that you have the latest support-v4.jar file. These JAR files do get updated between API versions. If you get NoClassDefErrors even though you've added the file, you may be missing the DrawerLayout class and other dependencies that were incorporated in API version 13. For best results, included the latest support-v4.jar from the most recent API version.

### Setup Drawer Layout Files

You also need to setup a view that will represent the individual drawer item in a layout file such as `res/layout/drawer_nav_item.xml`:

```xml
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@android:id/text1"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="@android:drawable/list_selector_background"
    android:gravity="center_vertical"
    android:paddingLeft="16dp"
    android:paddingRight="16dp"
    android:minHeight="?android:attr/listPreferredItemHeightSmall"
    android:textAppearance="?android:attr/textAppearanceListItemSmall"
    android:textSize="16sp"
    android:textColor="#111" />
```

Then in your `res/values/strings.xml` add the following:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="drawer_open">Open navigation drawer</string>
    <string name="drawer_close">Close navigation drawer</string>
</resources>
```

### Copy In FragmentNavigationDrawer

First, let's define our [FragmentNavigationDrawer.java](https://gist.github.com/nesquena/4e9f618b71c30842e89c) file by copying the text from the [linked gist](https://gist.github.com/nesquena/4e9f618b71c30842e89c). 

**UPDATE:** To use the gist as defined above with the support action bar library, use the code defined below:
```java
package com.codepath.examples.navdrawerdemo;

import java.util.ArrayList;

import android.support.v7.app.ActionBar;
import android.content.Context;
import android.os.Bundle;
import android.support.v4.app.ActionBarDrawerToggle;
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentActivity;
import android.support.v4.app.FragmentManager;
import android.support.v4.view.GravityCompat;
import android.support.v4.widget.DrawerLayout;
import android.support.v7.app.ActionBarActivity;
import android.util.AttributeSet;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.ListView;

public class FragmentNavigationDrawer extends DrawerLayout {
	  private ActionBarDrawerToggle drawerToggle;
		private ListView lvDrawer;
		private ArrayAdapter<String> drawerAdapter;
		private ArrayList<FragmentNavItem> drawerNavItems;
		private int drawerContainerRes;

		public FragmentNavigationDrawer(Context context, AttributeSet attrs, int defStyle) {
			super(context, attrs, defStyle);
		}

		public FragmentNavigationDrawer(Context context, AttributeSet attrs) {
			super(context, attrs);
		}

		public FragmentNavigationDrawer(Context context) {
			super(context);
		}
	    
		// setupDrawerConfiguration((ListView) findViewById(R.id.lvDrawer), R.layout.drawer_list_item, R.id.flContent);
		public void setupDrawerConfiguration(ListView drawerListView, int drawerItemRes, int drawerContainerRes) {
			// Setup navigation items array
			drawerNavItems = new ArrayList<FragmentNavigationDrawer.FragmentNavItem>();
			// Set the adapter for the list view
			drawerAdapter = new ArrayAdapter<String>(getActivity(), drawerItemRes, 
                            new ArrayList<String>());
			this.drawerContainerRes = drawerContainerRes; 
			// Setup drawer list view and related adapter
			lvDrawer = drawerListView;
			lvDrawer.setAdapter(drawerAdapter);
			// Setup item listener
			lvDrawer.setOnItemClickListener(new FragmentDrawerItemListener());
			// ActionBarDrawerToggle ties together the the proper interactions
			// between the sliding drawer and the action bar app icon
			drawerToggle = setupDrawerToggle();
			setDrawerListener(drawerToggle);
			// set a custom shadow that overlays the main content when the drawer
			setDrawerShadow(R.drawable.drawer_shadow, GravityCompat.START);
			// Setup action buttons
			getSupportActionBar().setDisplayHomeAsUpEnabled(true);
			getSupportActionBar().setHomeButtonEnabled(true);
		}

		// addNavItem("First", "First Fragment", FirstFragment.class)
		public void addNavItem(String navTitle, String windowTitle, Class<? extends Fragment> fragmentClass) {
			drawerAdapter.add(navTitle);
			drawerNavItems.add(new FragmentNavItem(windowTitle, fragmentClass));
		}
		
		/** Swaps fragments in the main content view */
		public void selectDrawerItem(int position) {
			// Create a new fragment and specify the planet to show based on
			// position
			FragmentNavItem navItem = drawerNavItems.get(position);
			Fragment fragment = null;
			try {
				fragment = navItem.getFragmentClass().newInstance();
				Bundle args = navItem.getFragmentArgs();
				if (args != null) { 
				  fragment.setArguments(args);
				}
			} catch (Exception e) {
				e.printStackTrace();
			}

			// Insert the fragment by replacing any existing fragment
			FragmentManager fragmentManager = getActivity().getSupportFragmentManager();
			fragmentManager.beginTransaction().replace(drawerContainerRes, fragment).commit();

			// Highlight the selected item, update the title, and close the drawer
			lvDrawer.setItemChecked(position, true);
			setTitle(navItem.getTitle());
			closeDrawer(lvDrawer);
		}
		

		public ActionBarDrawerToggle getDrawerToggle() {
			return drawerToggle;
		}

		
		private FragmentActivity getActivity() {
			return (FragmentActivity) getContext();
		}

		private ActionBar getSupportActionBar() {
			return ((ActionBarActivity) getActivity()).getSupportActionBar();
		}

		private void setTitle(CharSequence title) {
			getSupportActionBar().setTitle(title);
		}
		
		private class FragmentDrawerItemListener implements ListView.OnItemClickListener {		
			@Override
			public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
				selectDrawerItem(position);
			}	
		}
		
		private class FragmentNavItem {
			private Class<? extends Fragment> fragmentClass;
			private String title;
			private Bundle fragmentArgs;
			
			public FragmentNavItem(String title, Class<? extends Fragment> fragmentClass) {
				this(title, fragmentClass, null);
			}
			
			public FragmentNavItem(String title, Class<? extends Fragment> fragmentClass, Bundle args) {
				this.fragmentClass = fragmentClass;
				this.fragmentArgs = args;
				this.title = title;
			}
			
			public Class<? extends Fragment> getFragmentClass() {
				return fragmentClass;
			}
			
			public String getTitle() {
				return title;
			}
			
			public Bundle getFragmentArgs() {
				return fragmentArgs;
			}
		}
		
		private ActionBarDrawerToggle setupDrawerToggle() {
			return new ActionBarDrawerToggle(getActivity(), /* host Activity */
			this, /* DrawerLayout object */
			R.drawable.ic_drawer, /* nav drawer image to replace 'Up' caret */
			R.string.drawer_open, /* "open drawer" description for accessibility */
			R.string.drawer_close /* "close drawer" description for accessibility */
			) {
				public void onDrawerClosed(View view) {
					// setTitle(getCurrentTitle());
					// call onPrepareOptionsMenu()
					getActivity().supportInvalidateOptionsMenu(); 
				}

				public void onDrawerOpened(View drawerView) {
					// setTitle("Navigate");
					// call onPrepareOptionsMenu()
					getActivity().supportInvalidateOptionsMenu(); 
				}
			};
		}

		public boolean isDrawerOpen() {
			return isDrawerOpen(lvDrawer);
		}
		

}

```

### Define Fragments

Next, you need to define your fragments that will be displayed within the drawer. These can be any support fragments you define within your application.

### Setup Drawer in Activity

Next, let's setup a basic navigation drawer based on the following layout file which has the entire drawer setup in `res/layout/activity_main.xml`:

```xml
<com.codepath.examples.navdrawerdemo.FragmentNavigationDrawer
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    
    <!-- The main content view -->
    <FrameLayout
        android:id="@+id/flContent"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
    
    <!-- The navigation drawer -->
    <ListView android:id="@+id/lvDrawer"
        android:layout_width="240dp"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:choiceMode="singleChoice"
        android:divider="@android:color/darker_gray"
        android:dividerHeight="0dp"
        android:background="@android:color/background_light"
     />
</com.codepath.examples.navdrawerdemo.FragmentNavigationDrawer>
```

Now, let's setup the drawer in our activity:

```java
public class MainActivity extends FragmentActivity {
	private FragmentNavigationDrawer dlDrawer;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		// Find our drawer view
		dlDrawer = (FragmentNavigationDrawer) findViewById(R.id.drawer_layout);
		// Setup drawer view
		dlDrawer.setupDrawerConfiguration((ListView) findViewById(R.id.lvDrawer), 
                     R.layout.drawer_nav_item, R.id.flContent);
		// Add nav items
		dlDrawer.addNavItem("First", "First Fragment", FirstFragment.class);
		dlDrawer.addNavItem("Second", "Second Fragment", SecondFragment.class);
		dlDrawer.addNavItem("Third", "Third Fragment", ThirdFragment.class);
		// Select default
		if (savedInstanceState == null) {
			dlDrawer.selectDrawerItem(0);	
		}
	}


	@Override
	public boolean onPrepareOptionsMenu(Menu menu) {
		// If the nav drawer is open, hide action items related to the content
		if (dlDrawer.isDrawerOpen()) {
			menu.findItem(R.id.mi_test).setVisible(false);
		}
		return super.onPrepareOptionsMenu(menu);
	}

	@Override
	public boolean onCreateOptionsMenu(Menu menu) {
		MenuInflater inflater = getMenuInflater();
		inflater.inflate(R.menu.main, menu);
		return super.onCreateOptionsMenu(menu);
	}

	@Override
	public boolean onOptionsItemSelected(MenuItem item) {
		// The action bar home/up action should open or close the drawer.
		// ActionBarDrawerToggle will take care of this.
		if (dlDrawer.getDrawerToggle().onOptionsItemSelected(item)) {
			return true;
		}

		return super.onOptionsItemSelected(item);
	}

	@Override
	protected void onPostCreate(Bundle savedInstanceState) {
		super.onPostCreate(savedInstanceState);
		// Sync the toggle state after onRestoreInstanceState has occurred.
		dlDrawer.getDrawerToggle().syncState();
	}

	@Override
	public void onConfigurationChanged(Configuration newConfig) {
		super.onConfigurationChanged(newConfig);
		// Pass any configuration change to the drawer toggles
		dlDrawer.getDrawerToggle().onConfigurationChanged(newConfig);
	}

}
```

Now if you run your application, you should see the navigation drawer and be able to select between your fragments. For a more in-depth look at a navigation drawer with icons and sections, check out this [detailed navigation drawer tutorial](http://www.michenux.net/android-navigation-drawer-748.html). You might also want to check out this [drawer tutorial from AndroidHive](http://www.androidhive.info/2013/11/android-sliding-menu-using-navigation-drawer/).

## Alternative to Fragments

Although many navigation drawer examples show how fragments can be used with the navigation drawer, you can also use a RelativeLayout/LinearLayout if you wish to use the drawer as an overlay to your currently displayed Activity.  

Instead of &lt;FrameLayout&gt; you can substitute that for a &lt;LinearLayout&gt;

```java
<android.support.v4.widget.DrawerLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/drawer_layout">

        <LinearLayout
                android:id="@+id/content_frame"
                android:orientation="horizontal"
                android:layout_width="match_parent"
                android:layout_height="match_parent"/>

        <!-- The navigation drawer -->
        <ListView android:id="@+id/left_drawer"
                  android:layout_width="240dp"
                  android:layout_height="wrap_content"
                  android:layout_gravity="start"
                  android:choiceMode="singleChoice"
                  android:divider="@android:color/transparent"
                  android:dividerHeight="0dp"
                  android:background="#111"/>

</android.support.v4.widget.DrawerLayout>
```

Instead of this:

```java
// Insert the fragment by replacing any existing fragment
getFragmentManager().beginTransaction()
       .replace(R.id.content_frame, fragment)
       .commit();
```

You can instead use the `LinearLayout` container to inflate the Activity directly:

```java
LayoutInflater inflater = getLayoutInflater();
LinearLayout container = (LinearLayout) findViewById(R.id.content_frame);
inflater.inflate(R.layout.activity_main, container);
```

## Add Icons

In order to add icons adjacent to the title in navigation drawer, you can use a custom `ListView` along with a custom list adapter.

![NavIcons|250](http://i.imgur.com/U5AWxz4l.png)

## Download Nav Drawer Item icons

Download the following icons and add them to your drawable folders by right clicking the `res` folder > New > Other > Android > Android Icon Set. The follow the instructions on the wizard. This will automatically include correctly sized icons in each of your drawables folders.
* [ic_one](http://i.imgur.com/PfXVA78.png)
* [ic_two](http://i.imgur.com/xzIdYlo.png)
* [ic_three](http://i.imgur.com/k5o1mCJ.png)

## Update Drawer Layout File

Update `res/layout/drawer_nav_item.xml` to accommodate the drawer item icon.

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="48dp">
 
    <ImageView
        android:id="@+id/ivIcon"
        android:layout_width="25dp"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_marginLeft="12dp"
        android:layout_marginRight="12dp"
        android:layout_centerVertical="true" />
 
    <TextView
        android:id="@+id/tvTitle"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_toRightOf="@id/ivIcon"
        android:minHeight="?android:attr/listPreferredItemHeightSmall"
        android:textAppearance="?android:attr/textAppearanceListItemSmall"
        android:gravity="center_vertical"
        android:paddingRight="40dp"/>
   
</RelativeLayout>
```

## Add Model

Create a model class for your drawer icon. Name it `NavDrawerItem.java`.

```java
public class NavDrawerItem {    
    private String mTitle;
    private int mIcon;
     
    public NavDrawerItem(){}
 
    public NavDrawerItem(String title, int icon){
        this.mTitle = title;
        this.mIcon = icon;
    }
     
    public String getTitle(){
        return this.mTitle;
    }
     
    public int getIcon(){
        return this.mIcon;
    }
     
    public void setTitle(String title){
        this.mTitle = title;
    }
     
    public void setIcon(int icon){
        this.mIcon = icon;
    }     
}
```

## Create Custom Adapter

Next, create a custom adapter class for `ListView` which provides a custom layout for individual list item in the `ListView`.

```java
public class NavDrawerListAdapter extends BaseAdapter {

	private Context context;
    private ArrayList<NavDrawerItem> navDrawerItems;
     
    public NavDrawerListAdapter(Context context, ArrayList<NavDrawerItem> navDrawerItems){
        this.context = context;
        this.navDrawerItems = navDrawerItems;
    }
 
    @Override
    public int getCount() {
        return navDrawerItems.size();
    }
 
    @Override
    public Object getItem(int position) {       
        return navDrawerItems.get(position);
    }
 
    @Override
    public long getItemId(int position) {
        return position;
    }
 
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        if (convertView == null) {
            LayoutInflater mInflater = (LayoutInflater)
                    context.getSystemService(Activity.LAYOUT_INFLATER_SERVICE);
            convertView = mInflater.inflate(R.layout.drawer_nav_item, null);
        }
          
        ImageView imgIcon = (ImageView) convertView.findViewById(R.id.ivIcon);
        TextView txtTitle = (TextView) convertView.findViewById(R.id.tvTitle);
          
        imgIcon.setImageResource(navDrawerItems.get(position).getIcon());        
        txtTitle.setText(navDrawerItems.get(position).getTitle());
         
        return convertView;
    }

}
```

## Update FragmentNavigationDrawer

Now you should create an instance of `NavDrawerListAdapter`, add list items to it and assign this adapter to your Navigation Drawer ListView. Update the following code in `FragmentNavigationDrawer.java`.

```java
...
private ActionBarDrawerToggle drawerToggle;
private ListView lvDrawer;
private NavDrawerListAdapter drawerAdapter;
private ArrayList<NavDrawerItem> navDrawerItems;
private ArrayList<FragmentNavItem> drawerNavItems;    
private int drawerContainerRes;

// setupDrawerConfiguration((ListView) findViewById(R.id.lvDrawer), R.layout.drawer_list_item, R.id.flContent);
 	public void setupDrawerConfiguration(ListView drawerListView, int drawerItemRes, int drawerContainerRes) {
 		// Setup navigation items array
 		drawerNavItems = new ArrayList<FragmentNavigationDrawer.FragmentNavItem>(); 		
 		navDrawerItems = new ArrayList<NavDrawerItem>(); 				
 		this.drawerContainerRes = drawerContainerRes;  
 		// Setup drawer list view
 		lvDrawer = drawerListView; 
 		// Setup item listener
 		lvDrawer.setOnItemClickListener(new FragmentDrawerItemListener());
 		// ActionBarDrawerToggle ties together the the proper interactions
 		// between the sliding drawer and the action bar app icon
 		drawerToggle = setupDrawerToggle();
 		setDrawerListener(drawerToggle);
 		// set a custom shadow that overlays the main content when the drawer
 		setDrawerShadow(R.drawable.drawer_shadow, GravityCompat.START);
 		// Setup action buttons
 		getActionBar().setDisplayHomeAsUpEnabled(true);
 		getActionBar().setHomeButtonEnabled(true);
 	}

 	// addNavItem("First", R.drawable.ic_one, "First Fragment", FirstFragment.class)
 	public void addNavItem(String navTitle, int icon, String windowTitle, Class<? extends Fragment> fragmentClass) { 
 		// adding nav drawer items to array
 		navDrawerItems.add(new NavDrawerItem(navTitle, icon)); 
 		// Set the adapter for the list view
 		drawerAdapter = new NavDrawerListAdapter(getActivity(), navDrawerItems); 	
 		lvDrawer.setAdapter(drawerAdapter);
 		drawerNavItems.add(new FragmentNavItem(windowTitle, fragmentClass));
 	}

...

```

## Custom Background for Selected Item

To change the color of the selected item in your navigation drawer, you need to define layout drawable to  state the list item state when normal and pressed. It needs overall three xml files. One is for normal state, second is for pressed state and third one to combine both the layouts.

![Imgur](http://imgur.com/VDUQnKX)

Create a xml file under res > drawable folder named `list_item_bg_normal.xml` and paste the following code. (If you don’t see drawable folder, create a new folder and name it as drawable)

```xml
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
  <gradient
      android:startColor="#eeeeee"
      android:endColor="#eeeeee"
      android:angle="90" />
</shape>
```

Create another xml layout under res > drawable named `list_item_bg_pressed.xml` with following content.

```xml
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
  <gradient
      android:startColor="@android:color/holo_orange_light"
      android:endColor="@android:color/holo_orange_light"
      android:angle="90" />
</shape>
```

Create another xml file to combine both the drawable states under res > drawable named `list_selector.xml`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
 
    <item android:drawable="@drawable/list_item_bg_normal" android:state_activated="false"/>
    <item android:drawable="@drawable/list_item_bg_pressed" android:state_pressed="true"/>
    <item android:drawable="@drawable/list_item_bg_pressed" android:state_activated="true"/>
 
</selector>
```

Update `android:background` attribute of your `RelativeLayout` under `res/layout/drawer_nav_item.xml`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:background="@drawable/list_selector"
    android:layout_height="48dp">
 
    <ImageView
        android:id="@+id/ivIcon"
        android:layout_width="25dp"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_marginLeft="12dp"
        android:layout_marginRight="12dp"
        android:layout_centerVertical="true" />
 
    <TextView
        android:id="@+id/tvTitle"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_toRightOf="@id/ivIcon"
        android:minHeight="?android:attr/listPreferredItemHeightSmall"
        android:textAppearance="?android:attr/textAppearanceListItemSmall"
        android:gravity="center_vertical"
        android:paddingRight="40dp"/>
   
</RelativeLayout>
```


## References

* <http://www.michenux.net/android-navigation-drawer-748.html>
* <http://www.androidhive.info/2013/11/android-sliding-menu-using-navigation-drawer/>