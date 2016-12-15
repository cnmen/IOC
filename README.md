# IOC
##控件布局注入（非编译期，反射原理）
版本：2.3.5<br>
作者：西门提督<br>
日期：2016-12-14

###BaseActivity初始化操作
        public class BaseActivity extends AppCompatActivity {

            @Override
            protected void onCreate(@Nullable Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                IOCManager.initViewInject().injectView(this);
            }
        }

###BaseFragment初始化操作
        public class BaseFragment extends Fragment { // V4包

            @Nullable
            @Override
            public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
                return IOCManager.initViewInject().injectView(this, inflater, container);
            }
        }

###IOC的使用：
        @ContentView(R.layout.activity_login) // 注入布局（后续才起作用）
        public class LoginActivity extends BaseActivity {

            @InjectView(R.id.email)
            private AutoCompleteTextView mEmailView; // 任意控件

            @InjectView(R.id.password)
            private EditText mPasswordView; // 如果是自定义控件，这里写父类

            @InjectView(R.id.lv) // 如果R文件中有相同id，而布局中没有此id，会抛出异常
            private ListView lv;

            @InjectView(R.id.login_progress)
            private ProgressBar mProgressBarView;

            private List<String> list;

            @Override
            protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                initDatas();
            }

            private void initDatas() { // 模拟数据，这里就用最简单的ListView
                list = new ArrayList<>();
                for (int i = 0; i < 20; i++) {
                    list.add("test - " + i);
                }
                MyAdater adater = new MyAdater(this, list); // 不详细写了
                lv.setAdapter(adater);
            }

            /**
             * 点击事件注意几点：
             * 1.R.id.xxx 必须布局中存在
             * 2.修饰符目前必须private（不方便可以取消此限制）
             * 3.方法名可以任意取，但参数必须要有(View xxx)，否则抛出异常找不到方法
             * 4.做了防止误点连续触发控制，为：300毫秒
             * 5.与常规写法setOnClickListener事件不冲突
             * @param btn
             */
            @OnClick(R.id.email_sign_in_button)
            private void signInBtn(View btn) {
                String email = mEmailView.getText().toString().trim();
                String password = mPasswordView.getText().toString().trim();
                LogUtils.e(" >>> ", email + " / " + password);
                // do something here ...
            }

            /**
             * ListView条目点击事件
             * 1.R.id.xxx 必须布局中存在
             * 2.type值默认是OnClick事件，如不是必须填写
             * 3.修饰符目前必须private（不方便可以取消此限制）
             * 4.方法名可以任意取，但参数必须一一对应不容有错，否则抛出异常找不到方法
             * @param parent
             * @param view
             * @param position
             * @param id
             */
            @OnClick(value = R.id.lv, type = AdapterView.OnItemClickListener.class) // Update Your View's id
            private void onSomeThingItemClick(AdapterView<?> parent, View view, int position, long id) {
                LogUtils.e(" ItemClick>>> ", list.get(position));
                // do something here ...
            }

            /**
             * ListView条目长按事件
             * 1.R.id.xxx 必须布局中存在
             * 2.type值默认是OnClick事件，如不是必须填写
             * 3.修饰符目前必须private（不方便可以取消此限制）
             * 4.方法名可以任意取，但参数必须一一对应不容有错，否则抛出异常找不到方法
             * 5.注意方法返回是boolean
             * @param parent
             * @param view
             * @param position
             * @param id
             * @return
             */
            @OnClick(value = R.id.lv, type = AdapterView.OnItemLongClickListener.class) // Update Your View's id
            private boolean onSomeThingItemLongClick(AdapterView<?> parent, View view, int position, long id) {
                LogUtils.e(" ItemLongClick>>> ", list.get(position));
                // do something here ...
                return true;
            }

            /**
             * 长按事件
             * 1.R.id.xxx 必须布局中存在
             * 2.type值默认是OnClick事件，如不是必须填写
             * 3.修饰符目前必须private（不方便可以取消此限制）
             * 4.方法名可以任意取，但参数必须要有(View xxx)，否则抛出异常找不到方法
             * 5.注意方法返回是boolean
             * @param view
             * @return
             */
            @OnClick(value = R.id.email_sign_in_button, type = AdapterView.OnLongClickListener.class) // Update Your View's id
            private boolean onSomeThingLongClick(View view) {
                LogUtils.e(" LongClick>>> ");
                // do something here ...
                return true;
            }
        }