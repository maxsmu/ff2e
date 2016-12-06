## 排序

  quickSort(list) {
		if (list.length <= 1) {
			return list;
		}

		let pivotIndex = Math.floor((list.length - 1) / 2);
		let pivotVal = list[pivotIndex];
		let lessList = [];
		let moreList = [];

		list.splice(pivotIndex, 1);
		list.forEach(current => {
			+current < +pivotVal ? lessList.push(current) : moreList.push(current);
		});

		return this.quickSort(lessList).concat([pivotVal], this.quickSort(moreList));
	};
