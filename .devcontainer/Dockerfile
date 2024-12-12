# base镜像和版本
ARG VARIANT=22-bookworm
FROM node:${VARIANT}
# 设置一些局部变量,下面引用到了
ARG USERNAME=node
ARG NPM_GLOBAL=/usr/local/share/npm-global

# 把 NPM 存放全局 npm 包的目录 的路径加入到 PATH 中
ENV PATH=${NPM_GLOBAL}/bin:${PATH}

# 配置一个全局 npm 包安装目录,并确保其权限适配容器内用户的 UID 和 GID
# 确保当前用户可以使用全局npm的包; 而如果有新用户,只需要加入npm的组就行了
RUN \
    # 检查系统中是否已经存在名为 npm 的用户组;如果没有 npm 用户组,则创建一个系统级(-r)用户组npm; -r: GID会分配一个较小的值(通常小于 1000);这些组一般不会与普通用户关联
    if ! cat /etc/group | grep -e "^npm:" > /dev/null 2>&1; then groupadd -r npm; fi \
    # 将指定用户添加到 npm 用户组;-a:保留该用户原本的组成员关系
    && usermod -a -G npm ${USERNAME} \
    # 设置默认权限掩码(默认的权限减去umask);新创建的文件权限会是 664(文件)或 775(目录);这允许用户组成员拥有写权限; 默认的:666 文件: 目录:777
    && umask 0002 \
    # 创建存放全局 npm 包的目录
    && mkdir -p ${NPM_GLOBAL} \
    # 创建一个全局的 npm 配置文件
    && touch /usr/local/etc/npmrc \
    # 将全局 npm 目录和配置文件的拥有者设置为 ${USERNAME},所属组为 npm
    && chown ${USERNAME}:npm ${NPM_GLOBAL} /usr/local/etc/npmrc \
    # 设置 setgid 位,确保该目录下创建的文件或子目录继承用户组 npm;而不是创建者的默认组; 知识点:如果是g+s设置的是文件,则文件执行时,进程以文件所属的组权限运行
    && chmod g+s ${NPM_GLOBAL} \
    # 配置全局 npm 包安装目录为 ${NPM_GLOBAL};配置会写入到 /usr/local/etc/npmrc 中
    && npm config -g set prefix ${NPM_GLOBAL} \
    # 以 ${USERNAME} 用户执行同样的命令, 确保在用户环境中也生效;-g:配置会写入到 /usr/local/etc/npmrc 中
    && su ${USERNAME} -c "npm config -g set prefix ${NPM_GLOBAL}" \
    # 全局安装 eslint;一个 JavaScript 静态代码分析工具
    && su ${USERNAME} -c "umask 0002 && npm install -g eslint" \
    # 强制清理 npm 缓存目录中的文件
    && npm cache clean --force > /dev/null 2>&1

# 测试 build->context 的配置,确实需要 ".."
COPY server.js /test/sss.js
# 设置一个环境变量,一个表明这是devcontainer的容器的标志,以备后用
ENV DEVCONTAINER=true

# [Optional] Uncomment this section to install additional OS packages.
# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>

# [Optional] Uncomment if you want to install an additional version of node using nvm
# ARG EXTRA_NODE_VERSION=10
# RUN su node -c "source /usr/local/share/nvm/nvm.sh && nvm install ${EXTRA_NODE_VERSION}"

# [Optional] Uncomment if you want to install more global node modules
# RUN su node -c "npm install -g <your-package-list-here>"

# Install basic development tools
# RUN apt update && apt install -y less man-db sudo

# Ensure default `node` user has access to `sudo`
# ARG USERNAME=node
# RUN echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
#     && chmod 0440 /etc/sudoers.d/$USERNAME

