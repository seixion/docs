# 权限控制
## 前端控制

>前端主要使用vuex保存用户的权限信息，然后通过v-if 判断是否有权限，如果有权限就渲染这个dom元素。 我们以用户管理页面来讲解
    
>按钮v-if使用
    
        <template slot-scope="scope">
          <el-button v-if="sys_user_upd" size="small" type="success" @click="handleUpdate(scope.row)">编辑
          </el-button>
          <el-button v-if="sys_user_del" size="small" type="danger" @click="deletes(scope.row)">删除
          </el-button>
        </template>
 >变量初始化
 
    created() {
        this.getList();
        this.sys_user_add = this.permissions["sys_user_add"];
        this.sys_user_upd = this.permissions["sys_user_upd"];
        this.sys_user_del = this.permissions["sys_user_del"];
      }
      
 >从vuex 获取保存的permissions
 
    computed: {
        ...mapGetters(["permissions"])
      }
    permissions获取
    getUserInfo(state.token).then(response => {
        commit('SET_PERMISSIONS', data.permissions)
      })
      
## 后端控制

>注解@PreAuthorize，通过获取用户菜单列表，和请求的地址和请求方法对比判断有没有权限

        public boolean hasPermission(HttpServletRequest request, Authentication authentication) {
            Object principal = authentication.getPrincipal();
            List<SimpleGrantedAuthority> grantedAuthorityList = (List<SimpleGrantedAuthority>) authentication.getAuthorities();
            boolean hasPermission = false;
        
            if (principal != null) {
                if (CollectionUtil.isEmpty(grantedAuthorityList)) {
                    return hasPermission;
                }
        
                Set<MenuVo> urls = new HashSet<>();
                for (SimpleGrantedAuthority authority : grantedAuthorityList) {
                    urls.addAll(menuService.findMenuByRole(authority.getAuthority()));
                }
        
                for (MenuVo menu : urls) {
                    if (StringUtils.isNotEmpty(menu.getUrl()) && antPathMatcher.match(menu.getUrl(), request.getRequestURI())
                            && request.getMethod().equalsIgnoreCase(menu.getMethod())) {
                        hasPermission = true;
                        break;
                    }
                }
            }
            return hasPermission;
        }