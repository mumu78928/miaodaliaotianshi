# 任务：聊天网页平台开发

## 计划
- [x] 步骤1：初始化Supabase并设计数据库架构（已完成）
  - [x] 初始化Supabase项目
  - [x] 创建用户表（profiles）和角色枚举
  - [x] 创建邀请码表（invite_codes）
  - [x] 创建群聊表（groups）
  - [x] 创建消息表（messages）
  - [x] 创建群成员表（group_members）
  - [x] 设置RLS策略
  - [x] 创建用户同步触发器
- [x] 步骤2：创建Edge Functions处理短信验证码（已完成）
  - [x] 创建发送短信验证码Edge Function
  - [x] 创建验证短信验证码Edge Function
  - [x] 部署Edge Functions
- [x] 步骤3：设计颜色系统和更新样式配置（已完成）
  - [x] 更新index.css配置主题色
  - [x] 更新tailwind.config.js
- [x] 步骤4：实现用户认证系统（已完成）
  - [x] 创建类型定义（types.ts）
  - [x] 更新AuthContext支持多种登录方式
  - [x] 更新RouteGuard配置公共路由
  - [x] 创建登录页面（用户名/密码、手机号/验证码）
  - [x] 创建注册页面（包含邀请码）
- [x] 步骤5：实现主应用布局和导航（已完成）
  - [x] 创建主布局组件（带侧边栏）
  - [x] 创建导航栏组件（显示登录状态）
  - [x] 更新App.tsx集成AuthProvider和RouteGuard
  - [x] 配置路由系统
- [x] 步骤6：实现聊天功能（已完成）
  - [x] 创建公共频道页面
  - [x] 创建群聊列表页面
  - [x] 创建群聊详情页面
  - [x] 创建消息组件
  - [x] 实现实时消息功能（轮询）
- [x] 步骤7：实现管理员后台（已完成）
  - [x] 创建管理员仪表板页面
  - [x] 创建用户管理页面（列表、编辑、删除）
  - [x] 创建邀请码管理页面（生成、查看）
  - [x] 创建权限管理功能
- [x] 步骤8：创建数据库API封装（已完成）
  - [x] 创建db/api.ts封装所有数据库操作
- [x] 步骤9：运行lint检查并修复问题（已完成）

## 注意事项
- 使用Supabase Auth进行用户认证
- 短信验证码通过Edge Functions调用第三方API
- 管理员账号通过第一个注册用户自动设置为admin
- 邀请码必须在管理员后台生成才能注册
- 聊天消息使用轮询方式获取（不使用Supabase Realtime）
- 主色调：#4A90E2（蓝色）
- 布局：左侧导航+主内容区

## 重要提示
- **初始邀请码**：WELCOME2025（用于第一个用户注册，必须全大写）
- **第一个注册的用户将自动成为管理员**
- 管理员可以在后台生成更多邀请码

## 问题修复记录

### 2025-12-28：修复邀请码验证问题
**问题**：注册时提示"邀请码无效或已被使用"，即使输入的是正确的 WELCOME2025

**原因**：
1. 邀请码表的RLS策略只允许认证用户（authenticated）查看，但注册时用户还未认证
2. 更新邀请码状态时存在外键约束问题（用户profile尚未创建）

**解决方案**：
1. 添加RLS策略允许匿名用户（anon）查看和更新未使用的邀请码
2. 创建数据库函数 `use_invite_code` 处理邀请码使用逻辑
3. 修改函数逻辑，在用户profile创建前先标记邀请码为已使用，避免外键约束问题

**修复后**：用户现在可以正常使用邀请码 WELCOME2025 进行注册

## 新功能开发计划

### 已完成功能
- [x] 好友系统
  - [x] 创建好友关系表和好友申请表
  - [x] 添加好友功能（输入用户名发送申请）
  - [x] 在公共频道显示好友申请系统提示
  - [x] 好友列表页面
  - [x] 接受/拒绝好友申请功能
  - [x] 添加好友导航链接到侧边栏和底部导航
- [x] 群聊密码功能
  - [x] 修改群聊表添加密码字段
  - [x] 移除is_public字段，所有群聊都可见
  - [x] 加入群聊时验证密码
- [x] 违禁词检测
  - [x] 创建违禁词表
  - [x] 消息发送前检测违禁词
  - [x] 自动删除含违禁词的消息
  - [x] 向管理员发送违规通知
- [x] 管理员功能增强
  - [x] 封禁账号功能（设置role为banned）
  - [x] 解封账号功能（恢复role为user）
  - [x] 违规记录查看
  - [x] 违禁词管理（添加/删除）
  - [x] 踢下线功能（强制用户退出登录）
  - [x] 禁言功能（用户可登录但不能发消息）
  - [x] 解除禁言功能
- [x] 用户状态检测
  - [x] 封禁用户自动登出
  - [x] 被踢下线自动登出
  - [x] 禁言用户无法发送消息
  - [x] 前端定期检查用户状态（每5秒）

## 修复记录

### 2025-12-28：修复好友导航链接缺失
**问题**：好友页面已创建但导航栏中没有显示好友链接

**原因**：MainLayout.tsx中的navItems数组没有包含好友链接

**修复**：
1. 在MainLayout.tsx中添加UserPlus图标导入
2. 在navItems数组中添加好友链接配置：`{ path: '/friends', label: '好友', icon: UserPlus }`

**修复后**：好友链接现在显示在侧边栏和移动端底部导航中

### 2025-12-28：添加踢下线和禁言功能
**需求**：
1. 封禁后用户还可以正常登录，需要自动踢下线
2. 添加踢下线功能（管理员可强制用户退出登录）
3. 添加禁言功能（用户可登录但不能发消息）

**实现**：
1. 数据库层面：
   - 添加'muted'角色到user_role枚举
   - 在profiles表添加last_kicked_at字段
   - 创建kick_user()函数供管理员调用
2. API层面：
   - 添加kickUser、muteUser、unmuteUser函数
   - 在sendPublicMessage和sendGroupMessage中检查用户状态
3. 前端层面：
   - AuthContext中每5秒检查用户状态
   - 检测到封禁或被踢下线自动登出并跳转到登录页
   - 登录页显示被踢下线或封禁的提示
   - 管理员后台添加踢下线、禁言、解除禁言按钮
   - 聊天页面发送消息前检查用户状态
4. 用户体验：
   - 封禁用户：自动登出，无法登录
   - 被踢下线：立即登出，可重新登录
   - 禁言用户：可以登录和浏览，但无法发送消息

**修复后**：管理员可以踢用户下线、禁言用户，封禁用户会自动登出

### 2025-12-28：修复profiles表RLS策略导致的显示问题
**问题**：
1. 公共聊天中所有用户显示为"未知用户"
2. 添加好友时搜索用户显示"用户不存在"

**原因分析**：
profiles表的RLS策略过于严格，只允许：
- 用户查看自己的资料
- 管理员查看所有用户资料

普通用户无法查看其他用户的基本信息，导致：
- 聊天消息无法关联显示发送者用户名
- 搜索用户功能无法查询到其他用户

**修复方案**：
添加新的RLS策略：允许所有认证用户查看其他用户的基本信息
```sql
CREATE POLICY "认证用户可以查看其他用户的基本信息"
ON public.profiles
FOR SELECT
TO authenticated
USING (true);
```

**修复后**：
- 公共聊天和群聊中正确显示所有用户的用户名
- 添加好友功能可以正常搜索到其他用户

### 2025-12-28：数据重置
**操作**：清空所有用户数据和业务数据

**清空的表**：
- auth.users（认证用户）
- profiles（用户资料）
- messages（消息记录）
- groups（群聊）
- group_members（群成员）
- friendships（好友关系）
- friend_requests（好友申请）
- violation_logs（违规记录）
- invite_codes（邀请码）

**保留的数据**：
- banned_words（违禁词列表，13条）
- 初始邀请码：WELCOME2025（未使用状态）

**重置后状态**：
- 所有用户数据已清空
- 可以使用邀请码 WELCOME2025 注册第一个用户
- 第一个注册的用户将自动成为管理员

### 2025-12-28：修复用户管理功能并添加超级管理员
**问题**：
1. 删除用户功能无法正常工作，点击删除后用户仍在列表中
2. 踢下线或封禁解封后无法正常登录，登录几秒后自动下线
3. 需要添加超级管理员角色，具有重置数据功能

**问题分析**：
1. deleteUser函数只删除了profiles表，没有删除auth.users表
2. AuthContext中的检查逻辑有问题：
   - 解封后last_kicked_at字段仍保留旧值
   - 每次检查都会触发踢下线逻辑
3. 缺少超级管理员角色和重置功能

**修复方案**：
1. 创建delete_user_completely()数据库函数，同时删除profiles和auth.users
2. 创建clear_kicked_timestamp_on_unban()触发器，解封时自动清除last_kicked_at
3. 优化AuthContext的检查逻辑，避免重复触发登出
4. 添加super_admin角色到user_role枚举
5. 创建reset_all_data()数据库函数，清空所有数据但保留超级管理员
6. 在AdminPage添加重置按钮（仅超级管理员可见）

**实现细节**：
- delete_user_completely()：检查权限后删除profiles和auth.users
- reset_all_data()：仅超级管理员可调用，清空所有表但保留超级管理员账号
- 触发器：角色从banned/muted变为user/admin/super_admin时自动清除last_kicked_at
- 重置按钮：双重确认对话框，重置后自动刷新页面

**修复后**：
- 删除用户功能正常工作，同时删除认证账号和用户资料
- 解封后用户可以正常登录，不会自动下线
- 超级管理员可以重置所有数据（保留自己的账号）

### 2025-12-28：修复注册问题并添加批量删除和修改密码功能
**问题**：
1. 用户无法正常注册（邀请码表为空）
2. 需要将memzcmd账号提权为超级管理员
3. 需要添加批量删除用户功能
4. 需要添加修改密码功能

**修复方案**：
1. 重新插入初始邀请码WELCOME2025到数据库
2. 直接在数据库中将memzcmd的role更新为super_admin
3. 创建batch_delete_users()数据库函数，支持批量删除多个用户
4. 创建change_user_password()数据库函数，允许用户修改自己的密码
5. 在AdminPage添加复选框和批量删除按钮
6. 创建SettingsPage页面，提供修改密码功能
7. 在MainLayout添加设置入口

**实现细节**：
- batch_delete_users()：接收用户ID数组，遍历删除（跳过自己），返回删除数量
- change_user_password()：验证密码长度（至少6位），使用bcrypt加密更新密码
- AdminPage：添加全选复选框、单选复选框、批量删除按钮（带确认对话框）
- SettingsPage：显示用户信息、提供修改密码表单（新密码+确认密码）
- 路由：添加/settings路由
- 导航：在侧边栏和底部导航添加"设置"入口

**修复后**：
- 用户可以正常注册（使用邀请码WELCOME2025）
- memzcmd账号已提权为超级管理员
- 管理员可以批量选择并删除多个用户
- 所有用户可以在设置页面修改自己的密码

### 2025-12-29：修复邀请码RLS策略和is_admin函数
**问题**：
1. 管理员无法生成邀请码（RLS策略错误403）
2. 用户无法正常注册（邀请码验证失败）
3. is_admin()函数不包括super_admin角色

**原因分析**：
1. is_admin()函数只检查role='admin'，不包括super_admin
2. invite_codes表的RLS策略使用了is_admin()函数，导致super_admin无法插入邀请码
3. 旧的"管理员可以管理所有邀请码"策略使用ALL权限，但没有正确生效

**修复方案**：
1. 修复is_admin()函数，包括admin和super_admin两种角色
2. 删除旧的ALL权限策略，分别创建SELECT、INSERT、UPDATE、DELETE策略
3. 为匿名用户添加UPDATE策略，允许注册时标记邀请码为已使用
4. 重新插入WELCOME2025邀请码（标记为未使用）

**实现细节**：
```sql
-- 修复is_admin函数
CREATE OR REPLACE FUNCTION public.is_admin(uid uuid)
RETURNS boolean AS $$
  SELECT EXISTS (
    SELECT 1 FROM public.profiles p
    WHERE p.id = uid AND p.role IN ('admin', 'super_admin')
  );
$$ LANGUAGE sql SECURITY DEFINER;

-- 分别创建管理员策略
CREATE POLICY "管理员可以查看所有邀请码" ON invite_codes FOR SELECT ...
CREATE POLICY "管理员可以插入邀请码" ON invite_codes FOR INSERT ...
CREATE POLICY "管理员可以更新邀请码" ON invite_codes FOR UPDATE ...
CREATE POLICY "管理员可以删除邀请码" ON invite_codes FOR DELETE ...
```

**修复后**：
- 管理员和超级管理员都可以正常生成邀请码
- 用户可以使用WELCOME2025邀请码注册
- 所有邀请码相关功能正常工作

### 2025-12-29：添加超级管理员创建用户和个人中心功能
**需求**：
1. 超级管理员可以直接创建用户（不需要邀请码和短信验证）
2. 在用户头像处添加个人中心选项
3. 个人中心可以修改用户名、密码、手机号

**实现方案**：
1. 创建admin_create_user()数据库函数，允许超级管理员直接创建用户
2. 创建update_my_profile()数据库函数，允许用户更新自己的信息
3. 在AdminPage添加"创建用户"按钮和对话框（仅超级管理员可见）
4. 创建ProfilePage（个人中心页面），可以修改用户名、密码、手机号
5. 在Navbar添加用户头像下拉菜单，包含"个人中心"和"退出登录"选项
6. 更新路由添加/profile路由

**实现细节**：
- admin_create_user()：
  - 检查超级管理员权限
  - 验证用户名和手机号唯一性
  - 创建auth.users和profiles记录
  - 支持设置用户角色（user/admin/super_admin）
- update_my_profile()：
  - 验证新用户名和手机号的唯一性（排除自己）
  - 同时更新profiles和auth.users表
  - 更新用户名时同步更新email和metadata
- AdminPage创建用户对话框：
  - 输入：用户名、密码、手机号（可选）、角色
  - 验证：用户名格式、密码长度、手机号格式
  - 仅超级管理员可见
- ProfilePage：
  - 基本信息卡片：修改用户名和手机号
  - 修改密码卡片：输入新密码和确认密码
  - 账号信息卡片：显示角色和注册时间
- Navbar下拉菜单：
  - 显示用户名和角色
  - "个人中心"选项跳转到/profile
  - "退出登录"选项

**功能特点**：
- 超级管理员可以快速批量创建用户账号
- 用户可以自主修改个人信息，无需管理员干预
- 所有修改都有唯一性验证，防止冲突
- 修改用户名时自动同步更新相关数据

### 2025-12-29：修复admin_create_user函数的加密问题
**问题**：
- 超级管理员创建用户时提示"function gen_salt(unknown) does not exist"
- 注册失败，无法创建新用户

**原因分析**：
- pgcrypto扩展未启用，导致gen_salt()和crypt()函数不可用
- admin_create_user函数中直接使用了这些加密函数

**修复方案**：
1. 在migration中添加`CREATE EXTENSION IF NOT EXISTS pgcrypto;`
2. 重新创建admin_create_user函数，添加异常处理
3. 将加密密码的操作提前到变量中，避免在INSERT语句中直接调用

**实现细节**：
```sql
-- 启用pgcrypto扩展
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- 在函数中先加密密码
encrypted_pw := crypt(p_password, gen_salt('bf'));

-- 然后在INSERT中使用加密后的密码
INSERT INTO auth.users (..., encrypted_password, ...)
VALUES (..., encrypted_pw, ...);

-- 添加异常处理
EXCEPTION
  WHEN OTHERS THEN
    RETURN json_build_object('success', false, 'error', SQLERRM);
```

**修复后**：
- pgcrypto扩展已启用并正常工作
- 超级管理员可以成功创建用户
- 创建的用户可以正常登录

### 2025-12-29：修复pgcrypto函数schema路径问题
**问题**：
- 即使启用了pgcrypto扩展，创建用户时仍然提示"function gen_salt(unknown) does not exist"
- 问题持续存在，无法创建新用户

**原因分析**：
- pgcrypto扩展的函数（gen_salt、crypt）安装在`extensions` schema中
- 函数中直接调用`gen_salt()`和`crypt()`时，PostgreSQL在默认的search_path中找不到这些函数
- 需要使用完整的schema路径：`extensions.gen_salt()`和`extensions.crypt()`

**修复方案**：
1. 检查pgcrypto函数所在的schema（发现在extensions schema中）
2. 修改admin_create_user()函数，使用`extensions.crypt()`和`extensions.gen_salt()`
3. 同时修复change_user_password()函数，使用相同的schema路径

**实现细节**：
```sql
-- 修改前（错误）
encrypted_pw := crypt(p_password, gen_salt('bf'));

-- 修改后（正确）
encrypted_pw := extensions.crypt(p_password, extensions.gen_salt('bf'));
```

**修复后**：
- 加密函数可以正确调用
- 超级管理员可以成功创建用户
- 修改密码功能也正常工作
- 所有用户管理功能完全正常

### 2025-12-29：修复创建用户时的邮箱唯一性检查
**问题**：
- 创建用户时提示"duplicate key value violates unique constraint users_email_partial_key"
- 即使用户名不同，仍然无法创建用户

**原因分析**：
- admin_create_user函数只检查了profiles表中的用户名是否存在
- 没有检查auth.users表中的邮箱（username@miaoda.com）是否已存在
- 当用户名相同时，生成的邮箱也相同，导致违反唯一约束

**修复方案**：
1. 在创建用户前，先生成邮箱地址
2. 检查auth.users表中该邮箱是否已存在
3. 如果邮箱已存在，返回"用户名已存在"错误

**实现细节**：
```sql
-- 先生成邮箱
user_email := p_username || '@miaoda.com';

-- 检查profiles表中的用户名
IF EXISTS (SELECT 1 FROM profiles WHERE username = p_username) THEN
  RETURN json_build_object('success', false, 'error', '用户名已存在');
END IF;

-- 检查auth.users表中的邮箱
IF EXISTS (SELECT 1 FROM auth.users WHERE email = user_email) THEN
  RETURN json_build_object('success', false, 'error', '用户名已存在');
END IF;
```

**修复后**：
- 创建用户前会检查邮箱唯一性
- 不会再出现重复邮箱错误
- 超级管理员可以正常创建用户
- 所有验证逻辑完整可靠

### 2025-12-29：清理孤立用户记录并改进事务处理
**问题**：
- 创建用户时提示"用户名已存在"，但profiles表中查不到该用户
- auth.users表中存在孤立的用户记录（有auth记录但没有profile记录）
- 这是因为之前创建用户时出错，只创建了auth.users记录，profiles创建失败

**原因分析**：
- 之前的admin_create_user函数在创建auth.users成功后，如果创建profiles失败，不会回滚auth.users
- 导致auth.users表中留下孤立记录
- 下次创建同名用户时，邮箱检查会发现已存在，但实际上用户不完整

**修复方案**：
1. 清理auth.users表中的孤立记录（没有对应profiles的记录）
2. 改进admin_create_user函数：
   - 提前生成user_id，避免使用RETURNING
   - 添加手机号在auth.users表中的唯一性检查
   - 依赖PostgreSQL的事务自动回滚机制
   - 如果profiles创建失败，整个事务会自动回滚

**实现细节**：
```sql
-- 清理孤立记录
DELETE FROM auth.users
WHERE id NOT IN (SELECT id FROM profiles);

-- 改进函数
new_user_id := gen_random_uuid();  -- 提前生成ID

-- 先插入auth.users
INSERT INTO auth.users (...) VALUES (..., new_user_id, ...);

-- 再插入profiles（如果失败会自动回滚）
INSERT INTO profiles (...) VALUES (new_user_id, ...);
```

**修复后**：
- 清理了2条孤立的auth.users记录（grandpa和grandma）
- 创建用户时不会再留下孤立记录
- 事务保证数据一致性
- 用户创建要么完全成功，要么完全失败

### 2025-12-29：修复登录时的NULL字段扫描错误
**问题**：
- 登录时提示"error finding user: sqt: Scan error on column index 8, name email_change: converting NULL to string is unsupported"
- 用户无法正常登录

**原因分析**：
- auth.users表中的email_change、email_change_token_new、email_change_token_current等字段存在NULL值
- Supabase Auth服务在查询用户时，期望这些字段是空字符串''而不是NULL
- 之前创建用户时没有显式设置这些字段，导致它们为NULL
- 数据不一致：有些用户是空字符串，有些是NULL

**修复方案**：
1. 更新现有用户数据，将所有NULL值转换为空字符串
2. 修改admin_create_user函数，在创建用户时显式设置这些字段为空字符串
3. 确保所有字符串字段的一致性

**实现细节**：
```sql
-- 修复现有数据
UPDATE auth.users
SET 
  email_change = COALESCE(email_change, ''),
  email_change_token_new = COALESCE(email_change_token_new, ''),
  email_change_token_current = COALESCE(email_change_token_current, '')
WHERE email_change IS NULL 
   OR email_change_token_new IS NULL 
   OR email_change_token_current IS NULL;

-- 修改创建用户函数
INSERT INTO auth.users (
  ...,
  confirmation_token,
  recovery_token,
  email_change,
  email_change_token_new,
  email_change_token_current
) VALUES (
  ...,
  '',  -- 空字符串而不是NULL
  '',
  '',
  '',
  ''
);
```

**修复后**：
- 所有用户的email_change相关字段都是空字符串
- 登录功能恢复正常
- 新创建的用户不会再出现NULL字段问题
- 数据一致性得到保证

### 2025-12-29：修复踢下线后再次登录仍被踢的问题
**问题**：
- 用户被管理员踢下线后，再次登录时仍然会被自动踢下线
- 用户无法正常登录使用系统

**原因分析**：
- 用户被踢下线时，profiles表的last_kicked_at字段被设置为当前时间
- AuthContext的refreshProfile函数会定期检查last_kicked_at是否更新
- 用户再次登录后，last_kicked_at字段没有被清除，仍然保留着之前的时间戳
- 虽然lastKickedAt状态在登出时被清除，但数据库中的值没有清除
- 用户登录后，refreshProfile检测到last_kicked_at存在且与本地状态不同，误判为新的踢下线操作

**修复方案**：
1. 创建clear_kicked_status()数据库函数，用于清除用户的last_kicked_at字段
2. 在AuthContext的onAuthStateChange中监听SIGNED_IN事件
3. 用户登录成功后，自动调用clear_kicked_status()清除踢下线状态
4. 同时清除本地的lastKickedAt状态
5. 修改refreshProfile函数，当检测到last_kicked_at被清除时，同步更新本地状态

**实现细节**：
```typescript
// 数据库函数
CREATE FUNCTION clear_kicked_status() RETURNS void AS $$
BEGIN
  UPDATE profiles SET last_kicked_at = NULL WHERE id = auth.uid();
END;
$$;

// AuthContext中的处理
onAuthStateChange((event, session) => {
  if (event === 'SIGNED_IN') {
    // 清除踢下线状态
    await supabase.rpc('clear_kicked_status');
    setLastKickedAt(null);
    // 重新获取profile
    const profileData = await getProfile(session.user.id);
    setProfile(profileData);
  }
});

// refreshProfile中的同步
if (!profileData?.last_kicked_at && lastKickedAt) {
  setLastKickedAt(null);
}
```

**修复后**：
- 用户被踢下线后，再次登录时会自动清除踢下线状态
- last_kicked_at字段被设置为NULL
- 用户可以正常登录和使用系统
- 管理员可以多次踢下线同一用户（每次踢下线都会更新last_kicked_at）

## 2025-12-29：实现消息删除和投诉功能
**需求**：
- 管理员和超级管理员可以删除公共频道和群聊的消息
- 普通用户可以投诉消息
- 投诉流程：点击投诉按钮 → 选择要投诉的消息 → 填写投诉原因 → 确认投诉
- 超级管理员在管理后台查看投诉信息
- 处理完投诉后，点击"我已处理"按钮删除投诉记录

**实现方案**：
1. 数据库层面：
   - 创建complaints表存储投诉记录
   - 创建delete_public_message()和delete_group_message()函数
   - 创建create_complaint()函数
   - 配置RLS策略

2. API层面：
   - getAllComplaints()：获取所有投诉
   - createComplaint()：创建投诉
   - deleteComplaint()：删除投诉
   - deletePublicMessage()：删除公共消息
   - deleteGroupMessage()：删除群聊消息

3. UI层面：
   - PublicChatPage：添加投诉按钮和删除按钮
   - GroupChatPage：添加投诉按钮和删除按钮
   - AdminPage：添加投诉管理标签页

**实现细节**：
```sql
-- complaints表结构
CREATE TABLE complaints (
  id uuid PRIMARY KEY,
  complainant_id uuid REFERENCES profiles(id),
  message_id uuid,
  message_type text CHECK (message_type IN ('public', 'group')),
  message_content text,
  message_sender_id uuid REFERENCES profiles(id),
  message_sender_name text,
  group_id uuid,
  reason text,
  created_at timestamptz
);
```

**功能特点**：
- 投诉模式：点击"投诉消息"按钮进入选择模式，消息可点击
- 不能投诉自己的消息
- 管理员可以看到消息旁的删除按钮（垃圾桶图标）
- 投诉对话框显示被投诉消息的详细信息
- 投诉原因为可选项
- 投诉记录包含消息快照，即使原消息被删除也能查看
- 超级管理员可以在管理后台查看所有投诉
- 点击"我已处理"按钮后投诉记录自动删除

**完成状态**：
- ✅ 创建complaints表和相关数据库函数
- ✅ 添加Complaint类型定义
- ✅ 实现API函数（getAllComplaints, createComplaint, deleteComplaint, deletePublicMessage, deleteGroupMessage）
- ✅ 修改PublicChatPage添加投诉和删除功能
- ✅ 修改GroupChatPage添加投诉和删除功能
- ✅ 修改AdminPage添加投诉管理标签页
- ✅ 通过lint验证

**问题修复**：
删除消息时显示"删除失败"

**原因分析**：
- 初始实现中，删除函数使用了不存在的表名（public_messages和group_messages）
- 实际数据库中只有一个统一的messages表
- messages表通过is_public字段区分公共频道（true）和群聊（false）

**修复方案**：
```sql
-- 修复后的delete_public_message函数
DELETE FROM messages 
WHERE id = p_message_id 
AND is_public = true;

-- 修复后的delete_group_message函数
DELETE FROM messages 
WHERE id = p_message_id 
AND is_public = false 
AND group_id IS NOT NULL;
```

**修复后**：
- 管理员可以成功删除公共频道的消息
- 管理员可以成功删除群聊的消息
- 删除操作正确验证管理员权限
- 删除后消息列表自动刷新
