```js
/**
 * 获取分页按钮列表
 * @param {number} currentPage 当前页（从1开始）
 * @param {number} totalPage 总页数
 * @param {number} showPageCount 希望展示的按钮个数（至少6）
 * @returns {Array} 包含页码数字和 '...' 字符的数组
 */
function getPageList(currentPage, totalPage, showPageCount) {
  // 强制最小显示数量为6
  if (showPageCount < 6) {
    showPageCount = 6;
  }

  // 如果总页数不超过显示数量，直接返回所有页码
  if (totalPage <= showPageCount) {
    return Array.from({ length: totalPage }, (_, i) => i + 1);
  }

  const pages = [];
  // 总是显示第一页
  pages.push(1);

  // 中间区域需要显示的页码个数（扣除第一页和最后一页）
  const interiorCount = showPageCount - 2;
  // 尽可能让 currentPage 居中显示
  let left = currentPage - Math.floor(interiorCount / 2);
  let right = currentPage + Math.ceil(interiorCount / 2) - 1;

  // 如果左边越界，则调整右边界
  if (left < 2) {
    left = 2;
    right = left + interiorCount - 1;
  }
  // 如果右边越界，则调整左边界
  if (right > totalPage - 1) {
    right = totalPage - 1;
    left = right - interiorCount + 1;
  }

  // 如果左侧与第一页相差不大，则不需要省略号
  if (left === 2) {
    // 直接添加连续页码，不需要省略号
    for (let i = left; i <= right; i++) {
      pages.push(i);
    }
  } else if (left === 3) {
    // 如果只差一个页码，则把这个页码直接加入
    pages.push(2);
    for (let i = left; i <= right; i++) {
      pages.push(i);
    }
  } else {
    // 相差超过1个页码，显示省略号
    pages.push('...');
    for (let i = left; i <= right; i++) {
      pages.push(i);
    }
  }

  // 右侧部分：如果与最后一页相差不大，则不显示省略号
  if (right === totalPage - 1) {
    // nothing to do，最后一页后面没有其他页码
  } else if (right === totalPage - 2) {
    // 直接添加缺失的页码，不用省略号
    pages.push(totalPage - 1);
  } else {
    // 相差超过1个页码，添加省略号
    pages.push('...');
  }

  // 最后总是显示最后一页
  pages.push(totalPage);
  return pages;
}

// 测试示例
console.log(getPageList(1, 10, 6));   // 例如: [1, 2, 3, 4, 5, '...', 10]
console.log(getPageList(10, 10, 6));  // 例如: [1, '...', 6, 7, 8, 9, 10]
console.log(getPageList(5, 10, 6));   // 例如: [1, '...', 3, 4, 5, 6, 7, 10] 或者根据具体需求可能调整（详见下文）
console.log(getPageList(3, 10, 6));   // 例如: [1, 2, 3, 4, 5, '...', 10]
```
